---
title: Scale In Event
description: Recommendations how to handle a scale in event from an auto scaling group in Octopus Deploy.
position: 60
hideInThisSection: true
---

A scale in event is when virtual machines are removed from an auto-scaling group.  Just like a scale out event, a scale in event can be triggered automatically via a configured rule, or manually by a person.  When a scale in event occurs, the removed virtual machines will need to be removed from Octopus Deploy.

A scale in event will cause one of these scenarios.

- A server is registered with Octopus Deploy but no longer exists in the auto scaling group.  
- A server is deleted (manually or via a rule) in the middle of a deployment or runbook run.

Generally, a missing virtual machine registered as a deployment target is only a problem during a deployment or runbook run.  Depending how often you deploy, this might be a once a day problem, or a five times a day problem.  This is unlike workers, where a worker could be used hundreds of times a day in a variety of runbook runs or deployments.  