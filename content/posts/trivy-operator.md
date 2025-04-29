---
title: 'Trivy in Kubernetes'
date: 2024-10-01T17:15:18+02:00
draft: false
description: "How to setup the Trivy Operator in K8s"
summary: "A step-by-step guide to setup the Trivy Operator in Kubernetes."
slug: "trivy-operator"
tags: ["Kubernetes", "Trivy", "Trivy Operator"]
---

I am currently working with automated security testing to get control of the known vulnerabilities in our applications. As part of this, I am scanning a Kubernetes cluster and it's images, as well as application code. We want to cover the whole width, not only application code. Now, we look at a Infrastructure as Code (IaC) scanning tool, [Trivy](https://trivy.dev/).

Read more about Trivy in [my other post]({{< ref "trivy-info" >}}).

Trivy has a Kubernetes operator called [Trivy Operator](https://aquasecurity.github.io/trivy-operator/latest/). Advantages with using the Trivy Operator are ([source](https://www.aquasec.com/blog/vulnerability-scanning-trivy-vs-the-trivy-operator/)) :
- Trivy Operator does background scans continuously in the cluster
- Trivy CLI cannot detect changes of any resources running inside the cluster 
- Trivy Operator allows integrating with tools that can consume Kubernetes manifests as it produces [reports that are CRDs](https://aquasecurity.github.io/trivy-operator/latest/docs/crds/)
- Kubernetes best practice is to push information from within the cluster to tools outside rather than letting the tools pull data from the outside 

{{< alert >}}
One downside which is not in our scope, but is worth mentioning, is that the Trivy Operator does not scan prior to deployment. Hence, it can not be used as a quality control or quality gate. Therefore, you should still run Trivy CLI in your pipelines.
{{< /alert >}}


## Prereqs to follow this guide
* [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries)
* [Kubectl]()
* [kubecm](https://kubecm.cloud/), strictly not necessary, but I like this tool!
* [Helm](https://helm.sh/docs/intro/install/)
* [Trivy](https://aquasecurity.github.io/trivy/v0.50/docs/target/kubernetes/)
* An image that can be deployed to the cluster

## Info
The Trivy Operator continuously scans the Kubernetes cluster. From docs: 
> **The Operator does this by watching Kubernetes for state changes and automatically triggering security scans in response. For example, a vulnerability scan is initiated when a new Pod is created. This way, users can find and view the risks that relate to different resources in a Kubernetes-native way.**

## Step-by-step

### Setup local cluster
1. [Setup Flux with local registry]({{< ref "flux-with-local-registry">}})
   OR setup a simple cluster
   ```sh
   kind create cluster
   ```

2. Select local cluster (it is called kind-kind by default) with kubecm
   ```sh
   kubecm s kind-kind
   ```

3. For a more realistic environment, run a deployment
   ```sh
   k apply -f ~/git/testing/flux-image-updates/clusters/my-cluster/podinfo/podinfo-deployment.yaml 
   watch kubectl get pods
   ```

4. (optional) push image to the local registry and create deployment for it

    Trivy needs images to scan, but there are probably already other images in your cluster. E.g. from Flux or others.
    ```sh
    docker tag 0ac97f5bbbb5 localhost:5001/my-api:1.0.1
    docker push localhost:5001/my-api:1.0.1 
    ```

5. (optional) Check contents of registry

    Flux should automatically deploy new images when they get a new tag (according to the tag policy). To verify what's in the registry, you can curl it.
    ```sh
    ✗ curl -X GET http://localhost:5001/v2/_catalog            
    {"repositories":["my-api","hello-app","podinfo"]}
    **strong text**
    ```

    ```sh
    ✗ curl -X GET http://localhost:5001/v2/podinfo/tags/list
    {"name":"podinfo","tags":["5.0.7","5.0.3","5.0.5","5.0.0","5.0.4","5.0.6"]}
    ```


### (optional) Test Trivy Operator locally
https://aquasecurity.github.io/trivy-operator/latest/

I do this to get a better feeling of how Trivy works and how it should look in the cluster. You can install the Trivy Operator using a YAML manifest file, or as a Helm Chart. We will do the latter. Steps from the docs:

Option 1: Install from traditional Helm Chart repository

1. Add the Aqua chart repository:
```sh
helm repo add aqua https://aquasecurity.github.io/helm-charts/
helm repo update
```

2. Install the Helm Chart:
```sh
helm install trivy-operator aqua/trivy-operator \
  --namespace trivy-system \
  --create-namespace \
  --version 0.21.0
```

Installing the operator yields the following output:
```sh
 ✗    helm install trivy-operator aqua/trivy-operator \
     --namespace trivy-system \
     --create-namespace \
     --version 0.21.0
	 
NAME: trivy-operator
LAST DEPLOYED: Wed Mar 27 09:14:18 2024
NAMESPACE: trivy-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
You have installed Trivy Operator in the trivy-system namespace.
It is configured to discover Kubernetes workloads and resources in
all namespace(s).

Inspect created VulnerabilityReports by:

    kubectl get vulnerabilityreports --all-namespaces -o wide

Inspect created ConfigAuditReports by:

    kubectl get configauditreports --all-namespaces -o wide

Inspect the work log of trivy-operator by:

    kubectl logs -n trivy-system deployment/trivy-operator
```

Running these commands, we see that the operator is starting making reports. I am interested to see if the podinfo deployment has any reports. 

The instructions above are from the "Home" page of the docs, while there are also more options in the [Helm installation page](https://aquasecurity.github.io/trivy-operator/v0.19.0/getting-started/installation/helm/). 


### Play with the in-cluster API
... to get an overview of have the tool works. Is it even worth installing in the cluster?

Trivy creates several types of reports:
- VulnerabilityReport
- ConfigAuditReport
- ExposedSecretReport
- RbacAssessmentReport
- InfraAssessmentReport
- ClusterComplianceReport
- ClusterVulnerabilityReport
- SbomReport

#### Get an overview
To get an overview of all findings, we can use the reports as shown in the Helm Install output above:
```sh
✗ kubectl get vulnerabilityreports --all-namespaces -o wide
NAMESPACE            NAME                                                       REPOSITORY                           TAG                  SCANNER   AGE   CRITICAL   HIGH   MEDIUM   LOW   UNKNOWN
flux-system          replicaset-helm-controller-58d5cc6f5b-manager              fluxcd/helm-controller               v0.37.2              Trivy     92m   0          1      11       0     0
flux-system          replicaset-image-automation-controller-654dc4897-manager   fluxcd/image-automation-controller   v0.37.0              Trivy     93m   0          1      8        0     0
flux-system          replicaset-image-reflector-controller-8498c88d9-manager    fluxcd/image-reflector-controller    v0.31.1              Trivy     92m   0          0      9        0     0
...
```

```sh
✗ kubectl get configauditreports --all-namespaces -o wide
NAMESPACE            NAME                                               SCANNER   AGE   CRITICAL   HIGH   MEDIUM   LOW
default              replicaset-podinfo-5d869859bd                      Trivy     94m   0          2      3        9
default              service-kubernetes                                 Trivy     94m   0          0      0        0
my-api               replicaset-my-api-7cc565547                        Trivy     94m   0          1      2        9
flux-system          networkpolicy-allow-egress                         Trivy     94m   0          0      0        0
...
```

#### Dive deeper into findings
To checkout the findings, run the following commands
```sh
✗ kubectl describe vulnerabilityreport my-vulnerability-report -n default
✗ kubectl describe configauditreport my-configaudit-report -n default
```

My pod had the following HIGH finding:
```
    Category:     Kubernetes Security Check
    Check ID:     KSV118
    Description:  Security context controls the allocation of security parameters for the pod/container/volume, ensuring the appropriate level
 of protection. Relying on default security context may expose vulnerabilities to potential attacks that rely on privileged access.
    Messages:
      replicaset my-api-7cc565547 in my-api namespace is using the default security context, which allows root privileges
    Remediation:  To enhance security, it is strongly recommended not to rely on the default security context. Instead, it is advisable to exp
licitly define the required security parameters (such as runAsNonRoot, capabilities, readOnlyRootFilesystem, etc.) within the security context
.
    Severity:     HIGH
    Success:      false
    Title:        Default security context configured
```

#### Filter findings
There can be quite a lot of findings to filter through. Using `jq` is super nice for that. 

```sh
# Filtering config audit report
k get ConfigAuditReport -n namespace my-configaudit-report -ojson | jq '.report.checks[] | select(.severity=="CRITICAL")''
```

All in all.. This is not a great way to traverse the findings. Using some kind of vulnerability management tool is more or less a must imo. E.g. DefectDojo. I have written a blog post on a way of sending Trivy Operator findings to DefectDojo. 

### Review whether Trivy Operator is useful
Should you use the Trivy Operator? To decide on this, I've done some thinking and research.

- **Are the findings different from the CI pipeline scans?** - The findings are absolutely different. There were _a bunch_ more findings with Trivy Operator compared to the CI pipeline scan. I would do both scans (as long as the in-cluster scanning is useful).
- **Are the findings valuable for us?** - Many K8s platforms have at least two different types of consumers, the platform team and the teams owning applications set up.
  - **Now, for sure the platform team should have control of their findings**, but then how many of the resources are really under the platform team's control? Let's say you run `k get vulnerabilityreport --all-namespaces -o wide` and see that Calico has five different reports with four high criticality vulnerabilities and twentyfour medium criticality vulnerabilities. How does that help you? It is a nice way to evaluate the risk of using the tool, but most of us cannot do much more than that. Without some kind of vulnerability management tool, all you get is an overview where many of the findings are not actionable. It is nearly impossible to use it regularly. So then, should you disable scanning of the Calico namespaces and loose the visibility? Should you use a vulnerability management tool where you can manage the finding and show that you have reviewed it? Or should you just keep it the reports for an available overview? My conclusion is these findings are not useful without the possibility to mark them as accepted.  
  - **That brings us to the teams hosting their applications in the cluster**. For the findings to be valuable for them, some kind of reporting or vulnerability management tool might be necessary. Especially if they do not have access to query the cluster, have been taught to check the findings with kubectl and are OK with such a simple solution. I would not be OK with that. If there are no tools to handle this, maybe scanning those namespaces aren't valuable to you?
- **Which of the K8s resources should be scanned?** - The Trivy Operator does not have a way of caching images, it recognizes the running K8s resources and scans the image in that resource. So if a cron job or some automation is running and using the same image, it will scan it _every_ time. I would put those in a separate namespace and exclude scanning of those. 
- **Is image scanning done in pipelines?** - If so, is it worth it in the cluster? Well, the containers running in the cluster is what's running in production. This is where the real risk is. Also, teams can choose not to scan their images in their own pipelines. Cluster scanning is a way to enforce scans are performed. If the organization has some central data collection to keep track of the known vulnerabilities in the systems they own, then scanning images in the cluster or in your container repository might prove useful. 

### Setup Trivy Operator in production
#### Trivy Operator manifest files
https://aquasecurity.github.io/trivy/v0.50/tutorials/kubernetes/gitops/ 

Okay, so I think this will give value. Especially in a scenario where Kubernetes is used for standard applications in an organisation. Then we have centralized image scanning and can report known vulnerabilities without the teams having to set up anything themselves. 

So let's look into how to add the Trivy Operator to a cluster in production.

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: trivy-system
---
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: trivy-operator
  namespace: flux-system
spec:
  interval: 60m
  type: oci
  url: oci://ghcr.io/aquasecurity/helm-charts
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: trivy-operator
  namespace: trivy-system
spec:
  chart:
    spec:
      chart: trivy-operator
      version: 0.21.0
      sourceRef:
        kind: HelmRepository
        name: trivy-operator
        namespace: flux-system
  interval: 60m
  values:
    trivy:
      resources:
        limits: 
          memory: 1500M  # Default of ?? wasn't enough, causing OOMKilled workload containers
      ignoreUnfixed: true
    operator:
      scanJobsConcurrentLimit: 2  # Default of 10 used too much RAM at once for Nodes, causing OOMkilled workload containers
  install:
    crds: CreateReplace
    createNamespace: false
```

#### Configure Calico network policy
If you are using Calico or other network management tools and run the manifests above, you will most likely get the following error or something similar: 
`unable to run trivy operator: failed getting configmap: trivy-operator: Get "https://10.0.0.1:443/api/v1/namespaces/trivy-system/configmaps/trivy-operator": dial tcp 10.0.0.1:443: i/o timeout`. 

This means that you need to add a network policy. Trivy requires two network accesses:
- Access to the K8s API
- Access to the vulnerability database

Adhering to the principle of least privilege is quite hard here. In the network policy, only IP ranges can be set. Using Azure, one can set more tailored rules in front. However, those rules apply to the whole cluster, not one namespace or Kubernetes resource.  

Here is an example of a network policy for Calico to allow Trivy access to the K8s API:

```yaml
---
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: allow-trivy-operator-egress
  namespace: trivy-system
spec:
  types:
    - Egress
  egress:
    - action: Allow  # allow k8s API calls
      destination:
        services: 
          name: kubernetes
          namespace: default
    - action: Allow  # allow vulnerability database fetch
      destination:
        ports:
        - "443"
        nets:
        - 0.0.0.0/0
      protocol: TCP
```

To check the calico network policy when it has been applied:
```sh
calicoctl get networkpolicy allow-trivy-operator-egress -o yaml -n trivy-system --allow-version-mismatch
```


#### Tolerate node taints
https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

If your cluster has taints on nodes, you will see that the Trivy node collector isn't running correctly. You can check by first finding the node collector name (list all resources in the namespace and you will have it), and then run kubectl describe on it. The events will show what is wrong, e.g. `FailedScheduling` with the message `0/7 nodes are available: 1 node(s) had untolerated taint ....`.

```sh
k describe pod/node-collector-677f9fb5b8-jc6tw -n trivy-system
```

Find the taints on the nodes in your cluster: 
```sh
➜  ~ kubectl get nodes -o json | jq '.items[] | {name: .metadata.name, taints: .spec.taints}'
```

Then you can add it to the HelmRelease as values. Note that it is not common pod tolerations you must configure, but tell the Trivy Operator through `values.triyvOperator.scanJobTolerations` which tolerations the node-collector pod should have. 

<details><summary>What happens when setting common tolerations</summary>

Updating the tolerations, the node-collector pod didn't get them applied, only the operator pod (checking with `k describe pod/node-collector-some-id` and same for pod/trivy-operator). I found an [issue](https://github.com/aquasecurity/trivy-operator/issues/1659) and deleted all files to reset, but it didn't work. With some more research, I found that I must set the tolerations in the [Helm values](https://github.com/aquasecurity/trivy-operator/pull/1644#issuecomment-1884290908)

---

</details>

```yaml
...
  values:
    trivy:
      ignoreUnfixed: true
    trivyOperator:
			scanJobTolerations:
      - key: "key1"
        operator: "Equal"
        value: "value1"
        effect: "NoSchedule"
```

In version 0.21.1, it was supported to add tolerations to NodeCollector. At that point, we started getting the same toleration error messages and the NodeCollector timed out. Looking at the [HelmChart Artifact Hub](https://artifacthub.io/packages/helm/trivy-operator/trivy-operator?modal=values&path=nodeCollector.tolerations), we fixed by adding tolerations to the node collector as well: 

```yaml
...
  values:
    trivy:
      ignoreUnfixed: true
    trivyOperator:
			scanJobTolerations:
      - key: "key1"
        operator: "Equal"
        value: "value1"
        effect: "NoSchedule"
		nodeCollector:
		  - key: "key1"
        operator: "Equal"
        value: "value1"
        effect: "NoSchedule"
```

Done! 


#### Add image scanning
Trivy scans images by default. If it is not working, make sure you allow network to fetch the vulnerability database, as well as allowing network to fetch images from your repository. 
Maybe this will help:
* https://aquasecurity.github.io/trivy-operator/latest/docs/vulnerability-scanning/private-registries/
* https://aquasecurity.github.io/trivy-operator/v0.19.0/tutorials/private-registries/ 

## Create reports
Coming soon maybe.

## Grafana dashboard
Documentation at https://aquasecurity.github.io/trivy-operator/v0.22.0/tutorials/grafana-dashboard/#using-the-grafana-helm-chart.

This was an easy fix, however it took a little time to figure out how to use the gnetId to create image through Terraform. I made a PR to the Trivy docs, so it should be documented now.


## Useful commands
### Trivy Operator logs
From K8s:
```sh
k logs -n trivy-system deployment/trivy-operator
```

See all trivy-system resource (except from network policy):
```sh
k get all -n trivy-system
```

### Flux reconcile logs
```sh
flux logs --namespace flux-system --since=1h -f
```
```sh
flux logs --namespace flux-system --since=1h -f --kind=kustomization
```
```sh
flux logs --namespace flux-system --since=1h -f --kind=kustomization --name=trivy-prereqs
```
See new commits as they are detected: 
```sh
flux logs --namespace flux-system --since=1h -f --kind=gitrepository
```

### Delete/restart the operator
After doing lots of testing, you might want to delete the operator and install it again to see that a clean install works. You can do like this:
```sh
kubectl delete all --all -n trivy-system
```

Flux might automatically reinstall. If not, you can run
```sh
flux reconcile kustomization flux-system --with-source -n flux-system
```
or maybe
```sh
flux reconcile helmrelease trivy-operator -n trivy-system
```

Or just delete the files and see that the resources are gone, and then put them back. 

### Delete all reports
```sh
kubectl delete exposedsecretreport --all --all-namespaces
```
And then same for other reports, such as `vulnerabilityreport`.

## Improvements and further thoughts
There are several improvements to be done here. Here are some of my thoughts:
- Create reports, e.g. monthly reports or immediate alerts for critical findings
- Scan regularly
- Use hash of versions and do not scan that very image again every time it appears in the cluster. E.g. job images.

## Debugging

### SBOM decode error: failed to decode: multiple OS components are not supported
```json
{
  "level": "error",
  "ts": "2024-04-08T10:04:45Z",
  "logger": "reconciler.scan job",
  "msg": "Scan job container",
  "job": "trivy-system/scan-vulnerabilityreport-6cccfb67dd",
  "container": "k8s-cluster",
  "status.reason": "Error",
  "status.message": "2024-04-08T10:04:37.564Z\t\u001b[31mFATAL\u001b[0m\tsbom scan error: scan error: scan failed: failed analysis: SBOM decode error: failed to decode: failed to decode components: multiple OS components are not supported\n",
  "stacktrace": "github.com/aquasecurity/trivy-operator/pkg/vulnerabilityreport/controller.(*ScanJobController).completedContainers\n\t/home/runner/work/trivy-operator/trivy-operator/pkg/vulnerabilityreport/controller/scanjob.go:353\ngithub.com/aquasecurity/trivy-operator/pkg/vulnerabilityreport/controller.(*ScanJobController).SetupWithManager.(*ScanJobController).reconcileJobs.func1\n\t/home/runner/work/trivy-operator/trivy-operator/pkg/vulnerabilityreport/controller/scanjob.go:80\nsigs.k8s.io/controller-runtime/pkg/reconcile.Func.Reconcile\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.2/pkg/reconcile/reconcile.go:113\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Reconcile\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.2/pkg/internal/controller/controller.go:119\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).reconcileHandler\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.2/pkg/internal/controller/controller.go:316\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).processNextWorkItem\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.2/pkg/internal/controller/controller.go:266\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Start.func2.2\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.2/pkg/internal/controller/controller.go:227"
}
```

Haven't looked into this yet.


### scan error: unable to initialize an image scanner: remote error (image fetch)
This problem is because GETing the URL provides a bad response. 
```json
{
  "level": "error",
  "ts": "2024-04-23T03:07:55Z",
  "logger": "reconciler.scan job",
  "msg": "Scan job container",
  "job": "trivy-system/scan-vulnerabilityreport-7f665c795b",
  "container": "calico-windows-upgrade",
  "status.reason": "Error",
  "status.message": "2024-04-23T03:07:53.141Z\t\u001b[31mFATAL\u001b[0m\timage scan error: scan error: unable to initialize a scanner: unable to initialize an image scanner: 4 errors occurred:\n\t* docker error: unable to inspect the image (mcr.microsoft.com/oss/calico/windows-upgrade:v3.26.3): Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?\n\t* containerd error: containerd socket not found: /run/containerd/containerd.sock\n\t* podman error: unable to initialize Podman client: no podman socket found: stat podman/podman.sock: no such file or directory\n\t* remote error: GET https://mcr.microsoft.com/v2/oss/calico/windows-upgrade/manifests/v3.26.3: MANIFEST_UNKNOWN: manifest tagged by \"v3.26.3\" is not found; map[Tag:v3.26.3]\n\n\n",
  "stacktrace": "github.com/aquasecurity/trivy-operator/pkg/vulnerabilityreport/controller.(*ScanJobController).completedContainers\n\t/home/runner/work/trivy-operator/trivy-operator/pkg/vulnerabilityreport/controller/scanjob.go:353\ngithub.com/aquasecurity/trivy-operator/pkg/vulnerabilityreport/controller.(*ScanJobController).SetupWithManager.(*ScanJobController).reconcileJobs.func1\n\t/home/runner/work/trivy-operator/trivy-operator/pkg/vulnerabilityreport/controller/scanjob.go:80\nsigs.k8s.io/controller-runtime/pkg/reconcile.Func.Reconcile\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.3/pkg/reconcile/reconcile.go:113\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Reconcile\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.3/pkg/internal/controller/controller.go:119\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).reconcileHandler\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.3/pkg/internal/controller/controller.go:316\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).processNextWorkItem\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.3/pkg/internal/controller/controller.go:266\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Start.func2.2\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.3/pkg/internal/controller/controller.go:227"
}
```

I don't need Windows upgrades at all, so I am thinking of options to handle this:
* Disable scanning with Trivy
	* with label
	* target workload (turn off DaemonSet-reports)
	* namespace (turn off reports for all calico-system resources)
* Disable calico-windows-upgrade DaemonSet

Don't have a solution just yet. Didn't prioritise this because triggers this error is a deprecated feature that will be removed in the future.

### Scanning Windows images is not supported
Pretty self-explanatory. 
```json
{
  "level": "info",
  "ts": "2024-04-08T10:07:35Z",
  "logger": "reconciler.scan job",
  "msg": "Scan job container",
  "job": "trivy-system/scan-vulnerabilityreport-7f4d674d74",
  "container": "init",
  "status.reason": "Error",
  "status.message": "Scanning Windows images is not supported."
}
```

### Scan job - OOMKilled ✅
```json
{
  "level": "error",
  "ts": "2024-04-08T10:08:16Z",
  "logger": "reconciler.scan job",
  "msg": "Scan job container",
  "job": "trivy-system/scan-vulnerabilityreport-5c498d8bc6",
  "container": "prometheus",
  "status.reason": "OOMKilled",
  "status.message": "",
  "stacktrace": "github.com/aquasecurity/trivy-operator/pkg/vulnerabilityreport/controller.(*ScanJobController).completedContainers\n\t/home/runner/work/trivy-operator/trivy-operator/pkg/vulnerabilityreport/controller/scanjob.go:353\ngithub.com/aquasecurity/trivy-operator/pkg/vulnerabilityreport/controller.(*ScanJobController).SetupWithManager.(*ScanJobController).reconcileJobs.func1\n\t/home/runner/work/trivy-operator/trivy-operator/pkg/vulnerabilityreport/controller/scanjob.go:80\nsigs.k8s.io/controller-runtime/pkg/reconcile.Func.Reconcile\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.2/pkg/reconcile/reconcile.go:113\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Reconcile\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.2/pkg/internal/controller/controller.go:119\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).reconcileHandler\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.2/pkg/internal/controller/controller.go:316\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).processNextWorkItem\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.2/pkg/internal/controller/controller.go:266\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Start.func2.2\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.17.2/pkg/internal/controller/controller.go:227"
}
```

The default values for RAM wasn't enough. We gave a little more `request` and `limit` for the scanners and this solved the problem. In addition there were 10 reports generated simultanuously. We changed it to 2 to give the nodes a little room. It is shown under spec.values in [the manifest file](#trivy-operator-manifest-files).

### TOO MANY REQUESTS ✅
{{< alert >}}
This is an ongoing issue, so Trivy might solve this without us having to take action. However, it has been going on a while now. There are two options for this issue;
* Wait until the Trivy maintainers have fixed the issue
* Setup a mirror for the vulnerability database
{{< /alert >}}

At some point, we suddenly got the following error: 

```sh
2024-10-03T07:31:26Z	FATAL	Fatal error	init error: DB error: failed to download vulnerability DB: database download error: oci download error: failed to fetch the layer: GET https://ghcr.io/v2/aquasecurity/trivy-db/blobs/sha256:77a50f405854d311fdf062f2d7edf3c04c63e2f5d218751a29125431376757a1: TOOMANYREQUESTS: retry-after: 600.129µs, allowed: 44000/minute
```

I found out why this happens from a discussions thread in the Trivy repo: 
["This is happening just because Trivy has too many users and reached the rate limits. "](https://github.com/aquasecurity/trivy/discussions/7668#discussioncomment-10878985). Apparently, the GitHub container registry, ghcr.io, introduced rate limiting which ended up with this issue. Or, it is just Trivy that has gotten too many users.

So, in theory setting up cache for the vulnerability database should help. However, if we need to update the database often, it doesn't really help too much. It certainly doesn't solve the problem entirely. 

That leaves us two other options;
* **Setting up our own package registry to mirror the vulnerability database.** A comment on the option above states that this adds security risk. It definitely force us to maintain another thing and we risk not having the latest discovered vulnerabilities in our mirrored version. If the database doesn't get updated, we might just end up allowing another Log4j affect our systems. Anyways, I think this is a good option if this is still an issue next week. 
* If you are patient and OK with rerunning your failing pipelines, then a solution could be to **wait for the Trivy maintainers to fix the problem**. They are looking into the issue and have already pushed small improvements very quickly to remediate. I am sure they are looking into it and trying their best to fix the issue quickly.

Later, I found [this](https://github.com/aquasecurity/trivy/discussions/7699) announcement about the issue. 

## Resources
* https://aquasecurity.github.io/trivy/v0.50/docs/target/kubernetes/
* https://aquasecurity.github.io/trivy-operator/latest/
* https://www.aquasec.com/blog/vulnerability-scanning-trivy-vs-the-trivy-operator/
* https://github.com/aquasecurity/trivy/discussions/4905
* https://github.com/aquasecurity/trivy/discussions/4499
* https://aquasecurity.github.io/trivy/v0.17.2/private-registries/
* K8s Lens: https://docs.k8slens.dev/ 
* Lens extension: https://aquasecurity.github.io/trivy-operator/v0.10.1/tutorials/integrations/lens/
* https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/ 
* https://github.com/aquasecurity/trivy-operator/issues/1659
* 
