+++
title = 'DefectDojo with Trivy Operator cluster reports'
date = 2024-10-15T15:13:28+02:00
draft = false 
description = "Setting up DefectDojo and receiving data from Trivy Operator with trivy-dojo-report-operator."
summary = "A step-by-step guide to set up DefectDojo in a local Kind cluster and receive data from the Trivy Operator reports, using the trivy-dojo-report-operator."
slug = "defectdojo"
tags = ["k8s", "kind", "defectdojo", "trivy-dojo-report-operator", "trivy", "owasp"]
+++

[DefectDojo](https://www.defectdojo.org/) is an [OWASP](https://owasp.org/)
vulnerability management platform. It's free and open-source, with a paid SaaS
option available.

This guide shows you how to set up DefectDojo in a local Kind cluster and
automatically send Trivy Operator scan results to it using
[trivy-dojo-report-operator](https://github.com/telekom-mms/trivy-dojo-report-operator).

## Prerequisites
- A Kubernetes cluster (I'm using Kind)
- [Trivy Operator installed]({{< ref "trivy-operator" >}})
- Helm

{{< alert >}}
Using Kind? Check out my post on [setting up Kind with a local registry and Flux]({{< ref "flux-with-local-registry" >}}).
{{< /alert >}}

## Installing DefectDojo
### Docker Compose
DefectDojo is straightforward to run with Docker Compose—just clone the repo and
go. [See the official instructions](https://defectdojo.github.io/django-DefectDojo/getting_started/installation/).

However, for local cluster work, I recommend installing it directly in
Kubernetes to avoid networking headaches. When DefectDojo runs outside the
cluster, you'll need to deal with localhost routing between your machine and the
cluster. Much simpler to keep everything in Kubernetes.

Therefore, this guide will _not_ use Docker Compose.

### Kubernetes
{{< alert "link" >}}
[Official Kubernetes installation docs](https://github.com/DefectDojo/django-DefectDojo/blob/dev/readme-docs/KUBERNETES.md).
Fair warning: the docs are a bit dated and the setup isn't entirely
straightforward.
{{< /alert >}}

Since we can't simply add the Helm chart via Kubernetes manifests, here's what worked for me:

1. [Configure Kind for ingress](https://kind.sigs.k8s.io/docs/user/ingress/#create-cluster) and create the cluster. _Already done if you followed [my Kind setup post]({{< ref "flux-with-local-registry">}})._
2. [Install NGINX Ingress controller](https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx). _I used Flux to deploy it in my cluster._
3. Create the `defectdojo` namespace.
4. Run this bash script to install DefectDojo with Helm:
    ```sh
    #!/bin/sh

    # 0. Clone repo if not exists
    echo "[0] Cloning repo if not exists"
    if test -d /home/maritiren/git/testing/django-defectdojo; 
    then
    	echo "django-defectdojo exists, not cloning repo"
    else
    	echo "Repo doesn't exist, cloning..."
    	git clone https://github.com/DefectDojo/django-DefectDojo ~/git/testing/django-defectdojo
    fi

    ## 1. Fetch Helm repos
    echo "[1] Adding Defect Dojo Helm repo"
    helm repo add helm-charts 'https://raw.githubusercontent.com/DefectDojo/django-DefectDojo/helm-charts'
    helm repo update

    echo "[2] Adding Bitnami Helm repo"
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm repo update

    # 2. Update helm dependencies
    echo "[3] Updating Helm dependencies"
    cd /home/maritiren/git/testing/django-defectdojo
    helm dependency update ./helm/defectdojo

    # 3. Install helm chart
    echo "[4] Installing Defect Dojo with Helm"
    cd /home/maritiren/git/testing/django-defectdojo
    helm install \
      defectdojo \
      ./helm/defectdojo \
      -n defectdojo \
      --set django.ingress.enabled=true \
      --set django.ingress.activateTLS=false \
      --set createSecret=true \
      --set createRabbitMqSecret=true \
      --set createPostgresqlSecret=true \
      --set createMysqlSecret=false \
      --set createRedisSecret=false \
      --set "alternativeHosts={defectdojo-django.defectdojo.svc.cluster.local}"
      #> helm-install.out
      # setting alternative host for Trivy to send data

    # 4. set host value for defectdojo.default.minikube.local
    echo "[5] Adding host to /etc/hosts"
    if grep -Fxq "127.0.0.1 defectdojo.default.minikube.local" /etc/hosts
    then
    	echo "Already added to /etc/hosts. Skipping."
    else
    	echo "Adding host to /etc/hosts to resolve host value..."
    	echo "# Defect Dojo" | sudo tee -a /etc/hosts
    	echo "::1       defectdojo.default.minikube.local" | sudo tee -a /etc/hosts
    	echo "127.0.0.1 defectdojo.default.minikube.local" | sudo tee -a /etc/hosts
    fi

    # 5. print DefectDojo password
    echo "[6] DefectDojo admin password: $(kubectl \
      get secret defectdojo \
      --namespace=defectdojo \
      --output jsonpath='{.data.DD_ADMIN_PASSWORD}' \
      | base64 --decode)"

    echo "You should now (or as soon as the Kubernetes resources are finished setting up) be able to open http://defectdojo.default.minikube.local:8080."
    ```

## Sending Trivy Reports to DefectDojo
Now let's set up [trivy-dojo-report-operator](https://github.com/telekom-mms/trivy-dojo-report-operator)
to automatically forward Trivy Operator scans to DefectDojo.

{{< alert >}}
The [official installation docs](https://github.com/telekom-mms/trivy-dojo-report-operator?tab=readme-ov-file#installation-and-usage)
cover various methods, but we'll use Helm with Flux manifests here.
{{< /alert >}}

1. **Get your DefectDojo API key**: Click the person icon in DefectDojo, then "API v2 Key" → "Generate New Key".
2. **Configure the operator**: Add your API key and DefectDojo URL to the Helm values. Here's my Flux manifest:

    {{< alert >}}
**Important**: Use the internal cluster URL (`defectdojo-django.defectdojo.svc.cluster.local`), not the external one.
    {{< /alert >}}

    ```yaml
    # trivy-dojo-report-operator.yaml
    ---
    apiVersion: v1
    kind: Namespace
    metadata:
      name: defectdojo-report
    ---
    apiVersion: source.toolkit.fluxcd.io/v1
    kind: HelmRepository
    metadata:
      name: trivy-dojo-report-operator
      namespace: flux-system
    spec:
      interval: 60m
      url: https://telekom-mms.github.io/trivy-dojo-report-operator
    ---
    apiVersion: helm.toolkit.fluxcd.io/v2
    kind: HelmRelease
    metadata:
      name: trivy-dojo-report-operator
      namespace: defectdojo-report
    spec:
      chart:
        spec:
          chart: trivy-dojo-report-operator
          version: '0.6.2'
          sourceRef:
            kind: HelmRepository
            name: trivy-dojo-report-operator
            namespace: flux-system
      interval: 60m
      values:
        defectDojoApiCredentials:
          apiKey: "fa9b2d02...5a9c739b"
          url: "http://defectdojo-django.defectdojo.svc.cluster.local"  # For internal K8s routing 
        operator.trivyDojoReportOperator.env.defectDojoEngagementName: 'Trivy Operator'
        operator.trivyDojoReportOperator.env.defectDojoEvalTestTitle: "true"
        operator.trivyDojoReportOperator.env.defectDojoEvalProductName: "true"
        operator.trivyDojoReportOperator.env.defectDojoEvalProductTypeName: "true"
        operator.trivyDojoReportOperator.env.defectDojoTestTitle: 'f"{body[\'report'\][\'artifact\'][\'repository\']}:{body[\'report'\][\'artifact\'][\'tag\']}"'
        operator.trivyDojoReportOperator.env.defectDojoProductName: 'meta["labels"]["trivy-operator.resource.name"]'
        operator.trivyDojoReportOperator.env.defectDojoProductTypeName: 'meta["namespace"]'
      install:
        crds: CreateReplace
        createNamespace: true
    ```

And that's it! The operator will now automatically send new Trivy reports to
DefectDojo.

## Troubleshooting

### Connection Refused Error
If you see `Handler 'send_to_dojo' failed temporarily` with a connection error,
you're likely using the wrong URL.

{{< alert >}}
**Fix**: Use the internal cluster URL `http://defectdojo-django.defectdojo.svc.cluster.local` 
instead of the external one. Kubernetes routes traffic internally within the
cluster, so external URLs like `defectdojo.default.minikube.local` won't work
from pods.
{{< /alert >}}

<details>
<summary><strong>Debugging walkthrough</strong> (click to expand)</summary>

Here's the error you might encounter:
```sh
[2024-08-02 07:00:39,022] kopf.objects [ERROR] Handler 'send_to_dojo' failed temporarily: 
HTTPConnectionPool(host='defectdojo.default.minikube.local', port=80): 
Max retries exceeded with url: /api/v2/reimport-scan/
```

**Debugging approach**: First, verify the DefectDojo API is accessible:
```python
import requests

token = 'your_token'
url = 'http://defectdojo.default.minikube.local'
endpoint = 'api/v2/users'
headers = {'content-type': 'application/json',
            'Authorization': f"Token {token}"}
r = requests.get(f"{url}/{endpoint}", headers=headers, verify=True) # set verify to False if ssl cert is self-signed

for key, value in r.__dict__.items():
  print(f"'{key}': '{value}'")
  print('------------------'
```
This endpoint worked fine. Next, test `/api/v2/reimport-scan`, which requires a
report file. 

The operator uses [kopf](https://kopf.readthedocs.io/en/latest), a Python
operator framework. It has a handler that triggers when Trivy creates scan
reports:
```python
# handlers.py
@REQUEST_TIME.time()
@kopf.on.create(report.lower() + ".aquasecurity.github.io", labels=labels)
def send_to_dojo(body, meta, logger, **_):
   """
   The main function that creates a report-file from the trivy-operator vulnerabilityreport
   and sends it to the defectdojo instance.
   """
   ...
```

To see what's being sent to DefectDojo, I needed debug logs. Since the LOG_LEVEL
config wasn't working, I changed the operator code from `log.debug` to `log.info` 
to capture the request data. I also saved the report to `/tmp/report.json` in
the pod and copied it out:

```sh
kubectl cp -n defectdojo-report <pod-name>:/tmp/report.json ./report.json
```

Testing this request from my host machine worked perfectly—confirming it was a
networking issue, not a problem with the report format or API authentication.
   
**Testing from inside the pod**: To verify the operator couldn't reach DefectDojo,
I installed curl in the pod using `nsenter` from the host:
```sh
# Get the kopf process ID and enter its namespace
sudo nsenter -a -t $(pgrep kopf)

# Install curl
apt update && apt install -y curl
```

**The solution**: Kubernetes handles routing internally, so the operator needs
the internal cluster FQDN. Add it to DefectDojo's allowed hosts during
installation:
```sh
helm install \
  defectdojo \
  ./helm/defectdojo \
  -n defectdojo \
  --set django.ingress.enabled=true \
  --set django.ingress.activateTLS=false \
  --set createSecret=true \
  --set createRabbitMqSecret=true \
  --set createPostgresqlSecret=true \
  --set createMysqlSecret=false \
  --set createRedisSecret=false \
  --set "alternativeHosts={defectdojo-django.defectdojo.svc.cluster.local}" <----
```

Success! The operator now sends all Trivy reports to DefectDojo. 🎉

</details>

## Resources
- https://www.defectdojo.org/
- https://github.com/telekom-mms/trivy-dojo-report-operator

