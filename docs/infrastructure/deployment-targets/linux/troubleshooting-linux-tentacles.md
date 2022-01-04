---
title: Troubleshooting Linux Tentacles
description: How to troubleshoot problems with Octopus Linux Tentacles.
position: xx (?)
---

As with windows tentacles, all of the 'classic' problems of TCP networking: firewalls, proxies, timeouts, DNS issues, and so-on can affect Octopus Linux Tentacles. This guide will help to track down these issues specific for Linux Tentacles, such as when a previously working machine fails a health-check with errors.

Our guide for [installing Linux Tentacles](/docs/infrastructure/deployment-targets/linux/tentacle) covers how to install and configure new Linux Tentacles and check out our guide for [troubleshooting Windows Tentacles](/docs/infrastructure/deployment-targets/windows-targets/troubleshooting-tentacles.md) if you are using Windows based instances.    

This guide assumes you have already configured a Listening or Polling Tentacle service via the command:
```
/opt/octopus/tentacle/configure-tentacle.sh
```

### Restart the Tentacle service

Regardless of the status of the Tentacle service, often the best resolution is to simply restart any instance experiencing issues. 

The following command will restart all Tentacle services configured on an instance. To specify a singular instance, replace the '*' with the name of the Tentacle Service to restart:
```
/opt/octopus/tentacle/Tentacle service --instance='*' --restart
```



## Check the Tentacle service is running

Run the following command to see information about the tentacle service status. The name of the service will change depending on your configuration by default:
```
systemctl status Tentacle
```

**If the services is not running...** 

If the service is unable to be started via the command:
```
/opt/octopus/tentacle/Tentacle service --instance=<instance name> --start
```

It is likely that the service is crashing during startup; this can be caused by a number of things, most of which can be diagnosed from the Tentacle log files. Inspect these yourself, looking for FATAL or ERROR type messages, and either send the [log files](/docs/support/log-files.md) or extracts from them showing the issue to [Octopus Deploy Support](support@octopus.com) for assistance.

By default, the latest log file is located at:
```
/etc/octopus/<instance name>/Logs/OctopusTentacle.txt
```


**If the service is running**, continue to the next step.


### Check the Tentacle thumbprint

Verify that the thumbprint the Tentacle is trusting matches the Octopus Server thumbprint, as well as confirming the Server is trusting the correct Tentacle thumbprint.




## Check the IP address

Your Octopus Server or Tentacle Server may have multiple IP addresses that they listen on. For example, in Amazon EC2 machines in a VPN might have both an internal IP address and an external addresses using NAT. Octopus Server and Tentacle Server may not listen on all addresses; you can check which addresses are configured on the server by running `ipconfig /all` from the command line and looking for the IPv4 addresses.


## Check for zombie child processes locking TCP ports (Listening Tentacles)

If Tentacle fails to start with an error message like this: **A required communications port is already in use.**

The most common scenario is when you already have an instance of Tentacle (or something else) listening on the same TCP port. However, we have seen cases where there is no running Tentacle in the list of processes. In this very specific case it could be due to a zombie PowerShell.exe or Calamari.exe process that was launched by Tentacle that is still holding the TCP port. This can happen when attempting to cancel a task that has hung inside of Calamari/PowerShell. Simply rebooting the machine, or killing the zombie process will fix this issue, and you should be able to start Tentacle successfully.


## Check the server service account permissions



## Uninstall Linux Tentacle

If you get to the end of this guide without success, it can be worthwhile to completely remove the Tentacle configuration, data, and working folders, and then reconfigure it from scratch. This can be done without any impact to the applications you have deployed. Working from a clean slate can sometimes expose the underlying problem.

To uninstall (delete) a Tentacle instance run the `service --stop --uninstall` and then `delete-instance` commands first:

```
/opt/octopus/tentacle/Tentacle service --instance <instance name> --stop --uninstall
/opt/octopus/tentacle/Tentacle delete-instance --instance <instance name>
```

The working folders and logs can be deleted if they are no longer needed, depending on where you installed them, for instance:
```
# default locations:
# - installed directory:
cd /opt/octopus/tentacle

# - logs:
cd /etc/octopus

# - application directory:
cd /home/Octopus/Applications
```
