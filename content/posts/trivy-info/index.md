---
title: 'Trivy Infrastructure as Code (IaC) Scanning'
date: 2024-10-01T17:00:04+02:00
draft: false
description: "About the Trivy IaC scanning."
summary: "Contains several guides for applying Trivy to your project"
slug: "trivy"
tags: ["Trivy", "IaC scan", "quality"]
---

I am currently working with automated security testing to get control of the known vulnerabilities in our applications. As part of this, I am scanning a Kubernetes cluster and it's images, as well as application code. We want to cover the whole width, not only application code. Now, we look at a Infrastructure as Code (IaC) scanning tool, [Trivy](https://trivy.dev/).

![Trivy home page info screenshot](./trivy-info.png)

{{< alert >}}
Note! When it says "All-in-one" and "vulnerability scanning", it does not mean application code scanning. The focus is on IaC, so find something else in addition for your application code!_
{{< /alert >}}

I am very happy with Trivy scanning. It is a nice free tool that is easy to use in GitHub, GitLab, locally, and just wherever you want.

> I am just hoping that this is not another [Semgrep](https://semgrep.dev/), a nice free tool that ends up as a paid tool after it becomes popular.  

## Advantages
There are many pitfalls when writing IaC, and scanning it helps us in several ways. We don't need to **know everything**, and we don't need to pay constant attention to new vulnerabilities. We get **more control** of what we create, and create **more predictable applications**. We can be more confident on what we deliver when we run scans like this. 

## When & where to scan 
- At your local environment, before pushing to remote (VS Code extension, pre-commit hooks, etc.)
- As automatic PR review. Although, I only believe in using this as a quality gate as long as it is possible to buypass it. In case of critical scenarios, such as fixing down-time or a critical bug. 
- Regularly in the production environment, e.g. every saturday evening. 

## What scans to run
We should run different Trivy scans, and which will always differ from application to application. 
However, in general, I believe this scans should be run:
- **Configuration file scan**, to scan your IaC configuration files for known vulnerabilities
- **Secrets scan**, to check if there are any secrets present in your code
- **Dependency scan**, to scan your dependency versions for known vulnerabilities
- **Image scan**, to scan images. In my opinion, this should be done both on PR review and regularly in prod.
- **K8s cluster scan**, if you have a K8s cluster, this is a super way to scan images in the Kubernets cluster when they appear.

The three first scans can be done in Trivy filesystem scan, configuring it with `--scanners=misconfig,secret,vuln. I would do both the filesystem scan and image scan as CI checks in a PR review.

## Dockerfile scan vs image scan
> **Do we really need both? Yes! I asked my skilled colleagues this very questions, and can say that without doubt, you should do both!**

_This is section based on my colleague's explanation_.

Dockerfile scan checks the instructions used to build an image, such as the base image, package installations, config files and scripts. Also, whether you use best practices for versions, users with too high privileges or exposure of sensitive data, e.g. insecure mounting of fileshare. 

Image scanning finds known vulnerabilities in all installed packages, including the base image and dependencies that aren't directly visible in the Dockerfiel. Image scanning gives more of a depth of all the packages and third-party dependencies. E.g. using an image that is running an old version of Python with known vulnerabilities won't be found by the Dockerfile scan, but it will be discovered by the image scan. Only if the usage is explicitly written in the Dockerfile.

## Step-by-step guides for different scans
I am going to write different guides on applying Trivy in projects. Here are the ones I have so far:
* [Trivy in Kubernetes]({{< ref "trivy-operator" >}})
* Trivy in CI pipelines (coming soon)
