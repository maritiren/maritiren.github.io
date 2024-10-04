+++
title = 'Trivy CI Pipeline'
date = 2024-10-03T11:42:15+02:00
draft = true
+++




## Debugging

### go not scanned properly
Some pipelines got 44 findings, while other's had none. By looking at the pipeline logs, we figured that it wasn't detected properly that this was a Go repo. 

Running the commands locally also yields no findings (except one that is ignored with .trivyignore). With debug on, I found that Trivy is `Skipping vulnerability scan as no version is detected for the package name="my.project.com/some/module"`. 

```sh
2024-10-03T11:37:58+02:00	INFO	[gomod] Detecting vulnerabilities...
2024-10-03T11:37:58+02:00	DEBUG	[gomod] Scanning packages for vulnerabilities	file_path="go.mod"
2024-10-03T11:37:58+02:00	DEBUG	[gomod] Skipping vulnerability scan as no version is detected for the package name="my.project.com/some/module"
```

