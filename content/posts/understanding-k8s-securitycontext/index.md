+++
title = 'What is a SecurityContext in Kubernetes and Why it Matters?'
date = 2025-07-26T01:40:33+01:00
draft = false 
slug = "understanding-k8s-securitycontext"
tags = ["Kubernetes", "SecurityContext"]
categories = ["kubernetes"]
+++

When deploying applications in Kubernetes, security should never be an
afterthought. One of the most fundamental ways to harden your workloads is by
configuring the **SecurityContext**. This feature prevents common attack
vectors, is simple to implement, and provides defense in depth. It is reducing
risk without sacrificing functionality. 

Let's break down what it is and what a typical configuration looks like. This
article provides a baseline SecurityContext configuration that work for most 
applications.

## What is SecurityContext?

A SecurityContext is a set of security-related configurations applied to a
container or pod in Kubernetes. It defines how processes inside the container
interact with the host system, what permissions they have, and what restrictions
apply.

## Why Does This Matter?

By applying these configurations, you:

*   Reduce the attack surface
*   Prevent privilege escalation
*   Align with best practices for container usage

SecurityContext is a simple yet powerful tool for improving Kubernetes security
posture.

## Pod-level vs. Container-level 

The [Kubernetes documentation for SecurityContext](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
is excellent, but the scope can be a bit confusing. Most examples show `kind: Pod`,
which made me wonder: how does this work with Deployments? Since I rarely write
Pod manifests directly, it wasn't immediately clear.

Here's what you need to know:

### Two Levels of Configuration

SecurityContext can be configured at two levels:

**Pod-level** (`spec.securityContext`): Applies to all containers in the Pod.
Common settings include `fsGroup`, `runAsUser`, and `runAsGroup`.

**Container-level** (`spec.containers[].securityContext`): Applies to a specific
container. Settings like `readOnlyRootFilesystem`, `capabilities`, and
`allowPrivilegeEscalation` go here.

**Important**: Container-level settings override pod-level settings when both
*are defined.

### Example: Pod with Both Levels

Here's an example showing both levels:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:  # Pod-level: applies to all containers
    runAsUser: 10001
    runAsGroup: 10001
    fsGroup: 2000
  containers:
  - name: sec-ctx-demo
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    securityContext:  # Container-level: specific to this container
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
```

### Using SecurityContext in Deployments

Since Deployments create Pods, you configure SecurityContext in the Pod template.
Here's a recommended baseline configuration that works for most applications:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      securityContext:  # Pod-level
        runAsUser: 10001
        runAsGroup: 10001
      containers:
      - name: app
        image: my-app:v2.0.2
        securityContext:  # Container-level
          runAsNonRoot: true
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop: ["ALL"]
          seccompProfile:
            type: RuntimeDefault
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: cache
          mountPath: /app/cache
      volumes:
      - name: tmp
        emptyDir: {}
      - name: cache
        emptyDir: {}
```

Each configuration option is explained in detail in the sections below.

### Which Options Go Where?

Not all SecurityContext options can be set at both levels. Rather than listing them all here,
I recommend checking the official API documentation when configuring your SecurityContext:

For complete details, see:
- [Pod-level options](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.34/#podsecuritycontext-v1-core)
- [Container-level options](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.34/#securitycontext-v1-core)

## Common SecurityContext options explained

This section explains common SecurityContext options and when you should apply
them. Implementation of each config is thoroughly explained in my other article
[Implementing Kubernetes SecurityContext best practices using the Trivy VS Code extension]({{< ref "k8s-securitycontext-trivy-vscode" >}})

### readOnlyRootFilesystem

**What it does:** Makes the container's root filesystem read-only.

**Why it matters:** Prevents attackers from modifying system files, installing
malware, or persisting changes after compromise.

**When to use:** Always. Mount writable volumes only for directories that truly
need write access (like `/tmp`, caches, or logs). This follows the allow-listing
principle.

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

[See implementation guide →]({{< ref "k8s-securitycontext-trivy-vscode#readonlyrootfilesystem" >}})

### runAsNonRoot

**What it does:** Prevents the container from running as the root user (UID 0).

**Why it matters:** Running as root gives attackers full control if they compromise your container.
This is one of the most critical security settings.

**When to use:** Always. There are very few legitimate reasons to run containers as root.

```yaml
securityContext:
  runAsNonRoot: true
```

[See implementation guide →]({{< ref "k8s-securitycontext-trivy-vscode#runasnonroot" >}})

### runAsUser / runAsGroup

**What it does:** Explicitly sets the UID and GID for the container process.

**Why it matters:** `runAsNonRoot` only prevents root, but doesn't specify which user to use.

**When to use:** Always specify both. 

**Useful info:**
- The UID/GID must exist in your container image. This is either specified in
  your base image, or you must create the user and group yourself.
- Setting a high UID (10000+) avoids conflicts with system users on the host and
  aligns with security scanning tools like Trivy. This is usually not an issue,
  but why not just do it anyways?

```yaml
securityContext:
  runAsUser: 10001
  runAsGroup: 10001
```

[See implementation guide →]({{< ref "k8s-securitycontext-trivy-vscode#runasuser-runasgroup" >}})

### allowPrivilegeEscalation

**What it does:** Prevents processes from gaining more privileges than their
parent process.

**Why it matters:** Blocks exploits that try to escalate privileges using setuid
binaries or other mechanisms.

**When to use:** Always set to `false` unless you have a specific need for
privilege escalation (extremely rare).

```yaml
securityContext:
  allowPrivilegeEscalation: false
```

[See implementation guide →]({{< ref "k8s-securitycontext-trivy-vscode#allowprivilegeescalation" >}})

### capabilities

**What it does:** Controls which Linux capabilities the container has. Dropping `ALL` removes
unnecessary privileges.

**Why it matters:** Containers often inherit capabilities they don't need. Removing them reduces
what attackers can do.

**When to use:** Always drop `ALL` first. Add back only specific capabilities if your application
needs them (like `NET_BIND_SERVICE` for binding to ports below 1024).

```yaml
securityContext:
  capabilities:
    drop:
      - ALL
    # add:
    #   - NET_BIND_SERVICE  # Only if needed
```

**Pro tip:** Most applications don't need any capabilities if running on non-privileged ports.

[See implementation guide →]({{< ref "k8s-securitycontext-trivy-vscode#capabilities" >}})

### seccompProfile

**What it does:** Applies a seccomp (secure computing mode) profile to restrict system calls.

**Why it matters:** Limits which kernel system calls the container can make, reducing attack surface.

**When to use:** Use `RuntimeDefault` for most applications - it's a safe, general-purpose profile.
Custom profiles are rarely needed unless you have specific security requirements.

```yaml
securityContext:
  seccompProfile:
    type: RuntimeDefault
```

**Note:** This is increasingly becoming a default in Kubernetes, but explicitly setting it ensures
consistency across clusters.

[See implementation guide →]({{< ref "k8s-securitycontext-trivy-vscode#seccompprofile" >}})

## Conclusion

SecurityContext provides essential security controls that are simple to
implement. The baseline configuration shown in this article works for most
applications without special requirements.

The key settings to always include:
- `runAsNonRoot: true` and explicit `runAsUser`/`runAsGroup` with high UIDs
- `allowPrivilegeEscalation: false`
- `readOnlyRootFilesystem: true` (with volume mounts for writable directories)
- `capabilities.drop: ["ALL"]`
- `seccompProfile.type: RuntimeDefault`

While some options may seem complex at first, most are straightforward to apply.
Security scanners like Trivy will flag missing configurations, making it easy to
know what to add.

✅ *Next up: [How to implement these settings using Trivy in VS Code]({{< ref "k8s-securitycontext-trivy-vscode" >}})*

