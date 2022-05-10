---
title: Google cloud CLI scripts
description: Google cloud CLI Scripts allow you to manage your Google cloud resources as part of your deployment process.
position: 10
hideInThisSectionHeader: true
---

Octopus Deploy can help you run scripts on targets within Google Cloud Platform.

:::hint
Octopus can help you deploy your infrastructure and applications using a wide range of built-in steps, some of which use tools like the gcloud CLI that need to be available on the path of the machine that is executing the step. By default, these tools are not included in an Octopus installation, although some tooling may be included on [Cloud Dynamic Workers](/docs/infrastructure/workers/dynamic-worker-pools.md#available-dynamic-worker-images). It is best that you control the version of these tools - your deployments will likely rely on a specific version that they are compatible with to function correctly. The easiest way to achieve this is to use an [execution container](/docs/projects/steps/execution-containers-for-workers/index.md) for your steps.
If this is not an option in your scenario, we recommend that you provision your necessary client tools on your own worker.
:::

When executing a script against GCP, Octopus Deploy will automatically use your provided Google cloud account details to authenticate you to the target instance, or you can choose to use the service account associated with the target instance.

This functionality requires the Google cloud (gcloud) CLI to be installed on the worker.

## Run a gcloud script step {#RunningGcloudScript}

:::hint
The **Run gcloud in a Script** step was added in Octopus **2021.2**.
:::

Octopus Deploy provides a _Run gcloud in a Script_ step type, for executing script in the context of a Google Cloud Platform instance. For information about adding a step to the deployment process, see the [add step](/docs/projects/steps/index.md) section.

![](google-cloud-script-step.png "width=170")

![](google-cloud-script-step-body.png "width=500")

## Learn more

- How to create [Google cloud accounts](/docs/infrastructure/accounts/google-cloud/index.md)
