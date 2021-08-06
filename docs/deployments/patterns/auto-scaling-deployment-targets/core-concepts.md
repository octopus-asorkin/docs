---
title: Core Auto Scaling Concepts
description: Core concepts to understand when setting up an auto-scaling group with Octopus Deploy.
position: 10
hideInThisSection: true
---

The auto-scaling offerings from cloud providers introduce a lot of new terminology and concepts.  This page will explain some of those core concepts in relation to how it impacts Octopus Deploy.

# Scale Out Event

A scale out event is when virtual machines are added to an auto-scaling group.  It can be triggered automatically via a configured rule, or manually by a person.  When a scale out event occurs, the new virtual machines will need to be added to Octopus Deploy.

Link to registering virtual machines with Octopus Deploy.

# Scale In Event

A scale in event is when virtual machines are removed from an auto-scaling group.  Just like a scale out event, a scale in event can be triggered automatically via a configured rule, or manually by a person.  When a scale in event occurs, the removed virtual machines will need to be removed from Octopus Deploy.

Link to removing virtual machines from Octopus Deploy.

# Overprovisioning

Some cloud providers auto-scaling offerring, such as [Azure Virtual Machine Scale Sets](https://azure.microsoft.com/en-us/services/virtual-machine-scale-sets/) allow for overprovisioning.  When enabled, overprovisioning will create more virtual machines than asked for.  For example, a Azure Virtual Machine Scale Set is manually increased from 1 Virtual Machines to 10 Virtual Machines.  Instead of creating 9 virtual machines, 14 virtual machines are created.  After the virtual machines finish provisioning, the Virtual Machine Scale Set will remove the additional 5 virtual machines based on an internal algorithm.  

Typically, we've seen those additional 5 virtual machines register with Octopus Deploy prior to deletion.  When that occurs, overprovisioning is treated by Octopus Deploy as an extremely quick scale in event.  

# Custom or User Data

Custom or User data is custom information sent down to virtual machine while it is provisioning.  Each cloud provider processes custom data differently.  Azure will save all the data sent down into a specific file but will not do anything with it.  

# Launch Scripts

Launch scripts are custom scripts you provide to run when a virtual machine is created.  Generally, these are the scripts used to install the Octopus Tentacle.

- [AWS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
- [Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-windows)