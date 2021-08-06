---
title: Auto Scaling Recommendations
description: General recommendations when using any auto-scaling technology with Octopus Deploy.
position: 10
hideInThisSection: true
---

# Scaling In

Eventually you are going run into the following scenarios:

- A server is registered with Octopus Deploy but no longer exists in the auto scaling group.  
- A server is deleted (manually or via a rule) in the middle of a deployment or runbook run.

How the server is registered, as a deployment target or worker, affects the mitigation for those scenarios.