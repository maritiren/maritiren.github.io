---
title: 'Flux with Local Registry'
date: 2024-10-01T15:33:44+02:00
draft: false
description: "How to setup Flux in a Kind cluster with a local registry"
summary: "A step-by-step guide for setting up Flux in a Kind cluster with a local registry for automatic updates of Docker images."
slug: "flux-kind-localregistry"
tags: ["k8s", "kind", "local registry", "flux"]
---


This post show every step of setting up a Kind cluster with Flux for automatic image updates from a local registry. 

## Prerequisites
* [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries)
* Kubectl
* GitHub account


## 1. Setup Kind cluster with local registry
{{< alert >}}
The script below also includes Ingress config for hosting DefectDojo in your cluster. If you don't want to setup DefectDojo later, remove the whole `nodes:` section.
{{< /alert >}}

Find the script in the post [Kind Cluster with Local Registry]({{< ref "script-kind-cluster-with-local-registry" >}}). 
You may run this script even if you already have running local registry. It didn't do anything to my images at least. 
```sh
cd ~/git/flux-image-updates/clusters
./create-kind-cluster-with-registry.sh 
```


## 2. Run Flux
Following the Flux guide ["Automate image updates to Git"](https://fluxcd.io/flux/guides/image-update/#prerequisites), I setup everything as follows:

Start by adding your GitHub credentials as environment variables. 
PAT token can be found at [`Settings> Developer Settings> Tokens (classic)`](https://github.com/settings/tokens). 
It requires the `repo` scope.
```sh
export GITHUB_TOKEN=ghp_gCVsYEC...
export GITHUB_USER=maritiren
```

You can run this although you already have an existing GitHub repo for Flux (called flux-image-updates). This does not overwrite manifest files.
```sh
flux bootstrap github \
   --components-extra=image-reflector-controller,image-automation-controller \
   --owner=$GITHUB_USER \
   --repository=flux-image-updates \
   --branch=main \
   --path=clusters/my-cluster \
   --read-write-key \
   --personal
```


<a id="setup-secrets"></a>
## 3. Setup secrets for namespaces to fetch images
Add secret for the namespaces you use to fetch an image:
```sh
k create secret docker-registry regcred --docker-server="kind-registry:5000" --docker-username=myuser --docker-password=myuser -n flux-system
k create secret docker-registry regcred2 --docker-server="kind-registry:5000" --docker-username=myuser --docker-password=myuser -n my-api
```

Watch the image repositories to see status:
```sh
watch flux get image repository --all-namespaces
```

And then reconcile the image repositories:
```sh
flux reconcile image repository my-api -n my-api
flux reconcile image repository podinfo -n flux-system
```


<a id="push-image"></a>
## 4. Push image to local registry
```sh
docker tag 0ac97f5bbbb5 localhost:5001/my-api:1.0.0
docker push localhost:5001/my-api:1.0.0
```

podinfo: https://hub.docker.com/r/stefanprodan/podinfo 
```sh
docker pull stefanprodan/podinfo
```


<a id="check-reg-contents"></a>
## 5. (optional) Check contents of registry
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


<a id="cleanup"></a>
## Cleanup
Only delete the cluster:
```bash
kind cluster delete --name "kind-kind"
```


Deleting both the cluster and registry. 
_(From https://github.com/piyushjajoo/kind-with-local-registry-and-ingress/blob/master/destroy.sh)_:

```bash
#!/bin/sh
set -o errexit

# delete kind cluster
echo "deleting kind cluster"
kind delete cluster --name "kind-kind"

# delete registry
echo "deleting registry"
docker rm -f $(docker ps -a | grep registry | awk -F ' ' '{print $1}')
```

<a id="debugging"></a>
## Debugging
* [K8s can connect to local registry, but Flux can't](#item-zero)
* [K8s can't connect to local registry, but Flux can](#item-one)
* [Completely recreate deployment](#item-two)

<a id="item-zero"></a>
### K8s can connect to local registry, but Flux can't

{{< alert >}}
**Solution:** Use port 5000 in the Deployment manifest files and other places in K8s. Even thouhg the port of the registry is 5001.
{{< /alert >}}

This was one of the first huge issues I got while setting this up. I used so many hours debugging this. I cannot provide a complete overview of the debugging as I didn't take notes at that time. However, I remember what fixed access for Flux in the end. 

At first, the problem was that it didn't really connect to the local registry at all. At that point the error message was something like "Connection refused", so I thought I was going towards the local registry but had some authentication problem. **Turned out I wasn't even sending requests to the registry.** Sorry, I don't have the answer to this case here, probably changed some ports or DNS name of some sorts. 

Having the local registry receiving my requests, I started getting the error `http: server gave HTTP response to HTTPS client`. 

I tried a couple of things, on of them was to add credentials to the docker config file and restart dockerd. A simple mistake in the config was made, which in turn made dockerd go bananas. It restarted all the time, wouldn't allow us to reset and we ended up allowing more restarts in order to roll back the changes. This didn't solve anything either. 

Then in the end, it turned out the error was that we used port 5001, while Kubernetes was connecting to the registry using port 5000. In other words, **the solution was to use port 5000 in the Deployment manifest files and otherwise in Kubernetes**. We still aren't quite sure why, but assume this is due to how we connected the registry container and the cluster using `docker network connect`. Otherwise, from the outside of the cluster, we reach the registry using localhost:5001. PRETTY DARN CONFUSING! Localhost here and localhost there, K8s DNS name here. Good luck understanding this! 

<a id="item-one"></a>
### K8s can't connect to local registry, but Flux can

There are many (or few - depending on how you see things) options on what could be wrong. Here are a couple of different things that might help with this issue:

{{< alert >}}
**Solution:** What worked for me was to change the containerd configs in the cluster startup script.
{{< /alert >}}

#### Changing containerd configs
* https://github.com/kubernetes-sigs/kind/issues/2604#issuecomment-1041314277

Remove the existing config line and add this instead, in the [cluster startup script]({{< ref "script-kind-cluster-with-local-registry">}}). This makes some parts of the Kind script unuseful (the part of setting containerd settings in each node). You might change this either in cluster-config.yaml or in the Kind startup script.
```yaml
containerdConfigPatches:
- |-
  [plugins."io.containerd.grpc.v1.cri".registry]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors."kind-registry:5000"]
        endpoint = ["http://kind-registry:5000"]
```


#### Enable this in the Docker deamon (probably requires restart of deamon)
This didn't work really, but maybe? Maybe I didn't do it properly. Didn't work for me, though. 
```sh
✗ sudo cat << EOF > /etc/docker/daemon.json 
{
  "insecure-registries": ["localhost:5001"]
}
```

#### Set network connection with Docker
This should have been done in the script from Kind, but it was suggested as a solution. 
```sh
docker network connect kind kind-registry
```

<a id="item-two"></a>
### Completely recreate a deployment
```sh
✗ k delete deployment my-api -n my-api
✗ flux reconcile kustomization flux-system --with-source
```


## Resources
* https://fluxcd.io/flux/guides/image-update
* https://kind.sigs.k8s.io/docs/user/local-registry/
* https://hackernoon.com/kubernetes-cluster-setup-with-a-local-registry-and-ingress-in-docker-using-kind 
* https://stackoverflow.com/a/3175054
