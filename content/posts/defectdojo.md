+++
title = 'DefectDojo with Trivy cluster reports'
date = 2024-10-15T15:13:28+02:00
draft = true
description = "Setting up DefectDojo and receiving data from Trivy Operator with trivy-dojo-report-operator."
summary = "A step-by-step guide to set up DefectDojo in a local Kind cluster and receive data from the Trivy Operator reports, using the trivy-dojo-report-operator."
slug = "defectdojo"
tags = ["k8s", "kind", "defectdojo", "trivy-dojo-report-operator", "trivy", "owasp"]
+++

[DefectDojo](https://www.defectdojo.org/) is an [OWASP (Open Worldwide Application Security Project)](https://owasp.org/) project for vulnerability management. 

It is free, but also has paid SaaS solutions. 

First, the goal is to set up DefectDojo and receive data from Trivy reports in my Kubernetes cluster.
I also want to configure DefectDojo to fit a certain organization structure and see how SBOM fits in the platform. 
Then, I want to add scans from pipelines. 

## Prerequisites to follow these steps
- A local cluster (I use Kind)
- [The Trivy Operator running in the cluster]({{< ref "trivy-operator" >}})

{{< alert >}}
To understand my Kubernetes setup, checkout my posts about [running local Kind cluster with local registry and Flux]({{< ref "flux-with-local-registry" >}}).
{{< /alert >}}

## Installation of DefectDojo
### Docker Compose
It is very easy to run DefectDojo using Docker Compose. 
I just cloned the repo, ran it and it was all good. 
[Here are the instructions](https://defectdojo.github.io/django-DefectDojo/getting_started/installation/).

However, due to local networking, it is easier to setup with Kubernetes. 
Therefore, Kubernetes installation and configuring is the focus here.

### Kubernetes
{{< alert >}}
[Here's the Kubernetes installation documentation](https://github.com/DefectDojo/django-DefectDojo/blob/dev/readme-docs/KUBERNETES.md). Note that the Kubernetes documentation is old and not maintained. It wasn't straight forward to setup either.
{{< /alert >}}

A downside of the Kubernetes setup is that we cannot simply add the Helm chart using Kubernets manifest files. This is what I did in the end:
1. [Add cluster config to Kind](https://kind.sigs.k8s.io/docs/user/ingress/#create-cluster) and run cluster. _Note! This is already done if you setup your cluster like step 1 [in this post]({{< ref "flux-with-local-registry">}})_.
2. [Setup NGINX Ingress controller](https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx) and apply it. _I chose to just copy the files to create the Ingress controller with Flux in my local cluster._
3. Create and apply namespace `defectdojo`. I chose to do so as a manifest file.
4. Write and run bash script to setup DefectDojo with Helm locally:
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



## Send Trivy report data to DefectDojo
This involves setting up [trivy-dojo-report-operator](https://github.com/telekom-mms/trivy-dojo-report-operator) to automatically send report data created by the Trivy Operator to DefectDojo when they are created.


{{< alert >}}
Different ways of installing is described [here](https://github.com/telekom-mms/trivy-dojo-report-operator?tab=readme-ov-file#installation-and-usage). 
We are deploying with Helm, so none of the options in the docs correspond to these steps. See step 3 in this note for Helm setup. 
{{< /alert >}}

1. Clone repo, `git clone git@github.com:telekom-mms/trivy-dojo-report-operator.git`
2. Fetch the API key. Go to DefectDojo and click the person-icon, and then press "API v2 Key". Then "Generate New Key" and you got it!
3. Add the API key and the DefectDojo URL to the setup of your choice configuration and apply the changes. 
    I chose to apply the Helm chart as manifests:
    {{< alert >}}
Note that the URL is the internal cluster URL
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
        operator.trivyDojoReportOperator.env.defectDojoEvalEngagementName: "true"
        operator.trivyDojoReportOperator.env.defectDojoEvalProductName: "true"
        operator.trivyDojoReportOperator.env.defectDojoEvalProductTypeName: "true"
        operator.trivyDojoReportOperator.env.defectDojoEngagementName: 'body["report"]["artifact"]["tag"]'
        operator.trivyDojoReportOperator.env.defectDojoProductName: 'body[name]'
        operator.trivyDojoReportOperator.env.defectDojoProductTypeName: 'body["namespace"'
      install:
        crds: CreateReplace
        createNamespace: true
	```

4. Something isn't working as supposed to, with the following error message. It is a normal error message (as far as I know) when Python requests do not work correctly.
   ```sh
   [2024-08-02 07:00:39,022] kopf.objects         [ERROR   ] [flux-system/replicaset-image-reflector-controller-565565d549-manager] Handler 'send_to_dojo' failed temporarily: Other error occurred: HTTPConnectionPool(host='defectdojo.default.minikube.local', port=80): Max retries exceeded with url: /api/v2/reimport-scan/ (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7bb95574d430>: Failed to establish a new connection: [Errno 111] Connection refused')). Retrying in 60 seconds
   ```

## Debugging
### `Handler 'send_to_dojo' failed temporarily`
{{< alert >}}
I used the incorrect DefectDojo API URL in the config. The correct one was `http://defectdojo-django.defectdojo.svc.cluster.local`. It was previously `http://defectdojo.default.minikube.local` which works from outside the cluster. However, Kubernetes didn't route the request out of the cluster, only within.
{{< /alert >}}

Something isn't working as supposed to, with the following error message. It is a normal error message (as far as I know) when Python requests do not work correctly.
```sh
[2024-08-02 07:00:39,022] kopf.objects         [ERROR   ] [flux-system/replicaset-image-reflector-controller-565565d549-manager] Handler 'send_to_dojo' failed temporarily: Other error occurred: HTTPConnectionPool(host='defectdojo.default.minikube.local', port=80): Max retries exceeded with url: /api/v2/reimport-scan/ (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7bb95574d430>: Failed to establish a new connection: [Errno 111] Connection refused')). Retrying in 60 seconds
```

To debug this, I created a script to verify that the DefectDojo API could receive requests:
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
This simple endpoint worked just fine. Now I want to test the endpoint that the report operator is using, `/api/v2/reimport-scan`.  To do this, we need to attach a report file, and so we need to figure out the expected format. Therefore, I looked more into the report operator code. 
   
The report operator uses [kopf](https://kopf.readthedocs.io/en/latest), a Python framework for creating operators. The framework relies on creating handlers, and the report operator has a handler that fires when a new Trivy scan pod, named for instance `vulnerabilityreport.aquasecurity.github.io` is created:
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

The handler is called per report type. What's interesting here is to see what is sent as a file to DefectDojo.
   
To do this, I had to turn on DEBUG, but there was no config option. After struggling with the updated chart not working, I tried adding the LOG_LEVEL env var to the manifest and running manifests instead. Nothing really worked... Then I realized that either something magical is happening to the LOG_LEVEL (maybe through kopf is reading the environment variable and setting the level), or it is never set. So I tried to set it manually with `logger.setLevel("DEBUG")`. That didn't work either. I just need to see the object, so I ended up changing the log statement to `log.info` instead of `log.debug`. (Although I really wanted to do it the proper way, I guess I must realize when it is time to throw in the towel). The log now contains all the request data that is sent with the request. 
   
In addition, I stored the `report.json` to the filesystem in the pod, so that I could copy it to the host with `k cp -n defectdojo-report trivy-dojo-report-operator-operator-769978f8d5-rcklv:/tmp/report.json ./report.json`.
   
Now that I have everything I need to make the request from the host machine, I tried and it worked perfectly. Therefore, my suspicions (and fear) that this would be network related has been confirmed. 
   
#### Figure out network issue
To send requests from the Trivy Dojo Report Operator, we need to install cURL. However, we don't have sudo on the pod. Therefore, we did the following:
1. `pgrep kopf`. All processes in Kubernetes is visible to the host machine, so we can fetch the pid of the process. In our case, the kopf process.
2. `sudo nsenter -a -t $(pgrep kopf)`. We use the ID to enter the Docker container namespace with `nsenter`.
3. `apt update && apt install -y curl` 

OK, now we've got cURL on the pod. Let's go!

#### Solution
The problem was that Kubernetes didn't route the request out of the cluster, it deals with the routing internally. That means, the Trivy Dojo Report Operator couldn't find the FQDN of DefectDojo, as it is running in another namespace and didn't use the internal routing FQDN. It should be `defectdojo-django.defectdojo.svc.cluster.local`.  

DefectDojo doesn't allow this sort of host by default, so we must add it to an allow list upon start of DefectDojo.
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
  --set "alternativeHosts={defectdojo-django.defectdojo.svc.cluster.local}"  # <----
```

Woho! It worked! Trivy Dojo Report Operator is now sending all reports! 


## Resources
- https://www.defectdojo.org/
- https://github.com/telekom-mms/trivy-dojo-report-operator

