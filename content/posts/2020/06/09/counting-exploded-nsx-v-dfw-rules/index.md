---
title: "Counting Exploded NSX-v DFW Rules"
date: 2020-06-09
categories: 
  - "api"
  - "nsx"
  - "powernsx"
  - "scripts"
  - "vmware"

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true

cover:
  image: "images/count-dfw-rules-output.png"
  relative: true
  hiddenInSingle: true
  hiddenInList: false
---

When working with a customer recently, there was a question raised about how to calculate the actual number of NSX-v distributed firewall rules configured for a given VM on the dataplane of an ESXi host.

Whilst the short answer was to jump onto the console of the hypervisor or SSH into the host and look at the rules configured on the filter with the following command:

```
vsipioctl getrules -f <filtername>
```

Looks easy enough right? However, It didn't really work in this specific customers environment as SSH and console access to the hypervisors was managed by a 3rd party and hence it was impossible for the customer to run the above command.

So I put together a little Powershell script which leverages PowerNSX to grab the output of the above command to retrieve the DFW filters of a given VM and break down the actual number of rules that get programmed for each RuleId.

[https://github.com/dcoghlan/misc/tree/master/NSX-V/Count-DataPlane-Rules](https://github.com/dcoghlan/misc/tree/master/NSX-V/Count-DataPlane-Rules)

The script outputs the following:

- Detailed log file
- A copy of the output from the above vsipioctl command for each filter/vNic of the VM
- A CSV file which contains the count of rules per RuleId, along with the Total count of the RuleIds evaluated for each filter/vNic of the VM..

The script can be run in 3 different modes depending on what you are interested in:

- AllRules Mode: To evaluate all rules for all filters found on the specified VM

```
.\count-dataplane-rules.ps1 -Username admin -Password VMware1!VMware1! -Server 192.168.110.190  -VmName Dummy-011
```

- Section Mode: To evaluate all the rules within the section supplied (specified by Name) on all filters found on the specified VM

```
.\count-dataplane-rules.ps1 -Username admin -Password VMware1!VMware1! -Server 192.168.110.190  -VmName Dummy-011 -SectionName Moe
```

- RuleId Mode: To evaluate a specific rule (specified by rule id) on all filters found on the specified VM

```
.\count-dataplane-rules.ps1 -Username admin -Password VMware1!VMware1! -Server 192.168.110.190  -VmName Dummy-011  -RuleId 1009
```

This script should help you be able to understand any potential rule explosions that could be occurring within your environment.

And if you're interested in how you could reduce the number of rules configured on the dataplane, your could run the output of the vsipioctl getrules command that is saved in the output files through my DFW Optimizer scripts and see what it comes back with.

[https://github.com/dcoghlan/dfwoptimzer](https://github.com/dcoghlan/dfwoptimzer)

Let me know in the comments below if this is useful to anyone out there?
