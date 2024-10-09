+++
title = 'Trivy in CI Pipeline'
date = 2024-10-03T11:42:15+02:00
draft = true
slug = "trivy-pipeline"
tags = ["Trivy", "IaC scan", "CI", "pipeline"]
+++

I am currently working with automated security testing to get control of the known vulnerabilities in our applications. As part of this, I am scanning a Kubernetes cluster and it's images, as well as application code. We want to cover the whole width, not only application code. Now, we look at a Infrastructure as Code (IaC) scanning tool, [Trivy](https://trivy.dev/).

Read more about Trivy in [my other post]({{< ref "trivy-info" >}}).

## Debugging

### go not scanned properly
Some pipelines got 44 findings, while other's had none. By looking at the pipeline logs, we figured that it wasn't detected properly that this was a Go repo. 

Running the commands locally also yields no findings (except one that is ignored with .trivyignore). With debug on, I found that Trivy is `Skipping vulnerability scan as no version is detected for the package name="my.project.com/main/module"`. 

```sh
2024-10-03T11:37:58+02:00	INFO	[gomod] Detecting vulnerabilities...
2024-10-03T11:37:58+02:00	DEBUG	[gomod] Scanning packages for vulnerabilities	file_path="go.mod"
2024-10-03T11:37:58+02:00	DEBUG	[gomod] Skipping vulnerability scan as no version is detected for the package name="my.project.com/main/module"
```

Possibilitites:
* Set an older version? An idea from [this issue](https://github.com/aquasecurity/trivy/discussions/5744) stating it stopped working after version 0.45.1
* Maybe the cache isn't working properly

---

Trying an older version:
```sh
docker run -v $PWD:/src --workdir /src  aquasec/trivy:0.45.0 -d fs . --scanners config,vuln,secret --no-progress
...
2024-10-06T15:12:43.971Z	DEBUG	GOPATH (/root/go/pkg/mod) not found. Need 'go mod download' to fill licenses and dependency relationships
2024-10-06T15:12:43.979Z	DEBUG	OS is not detected.
2024-10-06T15:12:43.979Z	DEBUG	Detected OS: unknown
2024-10-06T15:12:43.979Z	INFO	Number of language-specific files: 1
2024-10-06T15:12:43.979Z	INFO	Detecting gomod vulnerabilities...
2024-10-06T15:12:43.979Z	DEBUG	Detecting library vulnerabilities, type: gomod, path: go.mod
...
Dockerfile (dockerfile)
=======================
Tests: 26 (SUCCESSES: 25, FAILURES: 1, EXCEPTIONS: 0)
Failures: 1 (UNKNOWN: 0, LOW: 1, MEDIUM: 0, HIGH: 0, CRITICAL: 0)
...
.trivycache/policy/content/commands/kubernetes/containerNetworkInterfaceFilePermissions.yaml (secrets)
======================================================================================================
Total: 1 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 1
```

Trying the newest version at this time, 0.56.0:
```sh
docker run -v $PWD:/src --workdir /src  aquasec/trivy:0.56.0 -d fs . --scanners misconfig,vuln,secret --no-progress
```

---

We cloned the Trivy repo and logged to check whether the packages has been scanned. On version 0.56.0, they are scanned even though the main module gets the message `Skipping vulnerability scan as no version is detected for the package name="my.project.com/main/module"`. However, when the scan finds all modules, it says `Number of language-specific files: N`, where N is more than just one.. This needs more investigation. 

Found [here](https://github.com/aquasecurity/trivy/blob/2c87f0cb794acd77446a273582ba1a45b9f18980/pkg/detector/library/detect.go#L32) in the Trivy code.

---

I think this might be a GitLab Runner issue. Perhaps the GitLab Runner cache. All local tests with and without Trivy cache is working as supposed to. Perhaps not, though. Still looking into it.

The results were inconsistent in GitLab. After clearing the pipeline cache, the pipeline findings are more consistent. However, they are not the same locally and in pipeline. Seems like due to recursive dependencies are added in the pipeline and not locally. Locally, I don't have the 
The "very" old results that had bout 35-44 findings, I think is because of GitLab Runner cache of the Go path.

Regarding the recursive dependencies, I see that the scan fetches packages from `$HOME/go/pkg/mod`. After running `go mod download`, I have other dependencies locally than what is scanned in the pipeline. That might cause different results.

---

I decided to try with GitHub Actions to see if it acts weird there as well. 

- The Trivy GH Action uses Trivy version 0.53.0 
- By default, it doesn't log the output from the Trivy run

---

Husk:
- Sjekk om det er Go cache
- 


## Resources
- https://www.aquasec.com/blog/devsecops-with-trivy-github-actions/
- https://github.com/aquasecurity/trivy/discussions/5744
- Trivy GitHub discussion: [Intermittent missing vulnerabilities from Trivy 0.55.0 #7585](https://github.com/aquasecurity/trivy/discussions/7585)
