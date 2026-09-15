---
title: "vRA 6.1 Hotfix for NSX"
date: 2015-01-07
categories: 
  - "vmware"
tags: 
  - "nsx"
  - "vcac"
  - "vmware"
  - "vra"

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
---

Whilst deploying a NSX 6.1 and vRA 6.1 lab for some testing, I came across a peculiar error in regards to deploying a 1-arm load balancer.

Essentially it wouldn’t work! When vRA was trying to execute the multi-machine blueprint, it would provision all the VMs correctly and then destroy them shortly afterwards. The logs would show the following error.

```
[Timezone] [Error]: VcoWorkflow 'Create Edge': Failed after 00:00:10.4947127, java.lang.RuntimeException: com.vmware.o11n.plugins.nsx.error.VsmException: VSM response error (10166): Invalid Configuration. To deploy NSX Edge appliances, at-least one Vnic must be configured and must be connected to a valid portgroup. (Workflow:Create edge / Scriptable task (item1)#57)
```

So after a bit of research, it turned out that you need to apply a hotfix for vRA 6.1 when integrated with NSX.

You can find the details of the KB article at the following link kb.vmware.com/kb/2088172

When applying the hotfix, I was unable to install some of the dll’s and I was scratching my head, as the instructions were quite simple.

**BEFORE** executing the following commands, you will need to right mouse click on each of the dlls listed and check on the General tab to see if Windows has “blocked” the dll as it believes its from an untrusted source. If the files are blocked, just click on the unblock button towards the bottom of the screen. Apparently Windows by default will block you from installing dlls which are downloaded from the internet and from untrusted sources.

Once the files were unbocked, all the commands in the hotfix instructions executed successfully.
