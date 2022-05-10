---
title: kubectl  
description: The kubectl utility is required by Octopus Deploy Kubernetes integration.  
position: 100
---

The [kubectl command-line tool](https://kubernetes.io/docs/reference/kubectl/overview/) is required by Octopus Deploy's Kubernetes features.

:::hint
Octopus can help you deploy your infrastructure and applications using a wide range of built-in steps, some of which use tooling like kubectl that need to be available on the path of the machine that is executing the step. By default, these tools are not included in an Octopus installation, although some tooling may be included on [Cloud Dynamic Workers](/docs/infrastructure/workers/dynamic-worker-pools.md#available-dynamic-worker-images). It is best that you control the version of these tools - your deployments will likely rely on a specific version that they are compatible with to function correctly. The easiest way to achieve this is to use an [execution container](/docs/projects/steps/execution-containers-for-workers/index.md) for your steps.
If this is not an option in your scenario, we recommend that you provision your necessary client tools on your own worker.
:::

By default, Octopus assumes `kubectl` is available in the PATH environment variable. A specific location for `kubectl` can be supplied by setting a `Octopus.Action.Kubernetes.CustomKubectlExecutable` variable in the Octopus project (an example value is `c:\tools\kubectl\version\kubectl.exe`). 
