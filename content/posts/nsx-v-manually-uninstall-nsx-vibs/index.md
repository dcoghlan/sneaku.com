---
title: "NSX-v: Manually uninstall NSX VIBs"
date: 2016-01-21
categories: 
  - "nsx"
  - "vmware"
tags: 
  - "nsx"
  - "vmware"
coverImage: "uninstall.jpg"
---

Due to a recent lab rebuild, I needed to manually remove the NSX VIBs that were installed on my ESXi host, and thought I would document the process so I can remember it in the future.

> _Removing the NSX VIBs from a host manually does not remove the configuration stored on the NSX Manager, so care needs to be taken when doing this._

> _Depending on the version of NSX-v installed, the number of NSX VIBs installed will vary._

For NSX-v versions prior to 6.2.x there are 3 VIBs as listed below

- esx-dvfilter-switch-security
- esx-vxlan
- esx-vsip

For NSX-v versions 6.2.x and above, the esx-dvfilter-switch-security VIB (as well as the new traceflow VIB) was rolled into the esx-vxlan VIB leaving only 2 VIBs.

- esx-vxlan
- esx-vsip

The steps to manually uninstall the appropriate NSX-v VIBs are as follows:

Check that the VIB you want to remove is actually installed on the ESXi host.

```
esxcli software vib get --vibname <name of VIB>
```

```
~ # esxcli software vib get --vibname esx-dvfilter-switch-security
 [NoMatchError]
 No VIB matching VIB search specification 'esx-dvfilter-switch-security'.
 Please refer to the log file for more details.
~ #
~ # esxcli software vib get --vibname esx-vxlan
VMware_bootbank_esx-vxlan_5.5.0-0.0.2983935
   Name: esx-vxlan
   Version: 5.5.0-0.0.2983935
   Type: bootbank
   Vendor: VMware
   Acceptance Level: VMwareCertified
   Summary: Vxlan and host tool
   Description: This package loads module and configures firewall for vxlan networking.
   ReferenceURLs: 
   Creation Date: 2015-08-14
   Depends: esx-base >= 5.5, esx-base <= 6.0.0, nsx-api <= 1, vmkapi_2_2_0_0
   Conflicts: 
   Replaces: esx-traceflow, esx-dvfilter-switch-security
   Provides: com.vmware.vxlan = 1.0.0.0-nsx, com.vmware.switchsecurity = 1.0.0.0, com.vmware.traceflow = 1.0.0.0, com.vmware.bfd = 1.0.0.0
   Maintenance Mode Required: False
   Hardware Platforms Required: 
   Live Install Allowed: True
   Live Remove Allowed: False
   Stateless Ready: True
   Overlay: True
   Tags: 
   Payloads: esx-vxla
~ #
~ # esxcli software vib get --vibname esx-vsip
VMware_bootbank_esx-vsip_5.5.0-0.0.2983935
   Name: esx-vsip
   Version: 5.5.0-0.0.2983935
   Type: bootbank
   Vendor: VMware
   Acceptance Level: VMwareCertified
   Summary: vsip module
   Description: This package contains DFW and NetX data and control plane components.
   ReferenceURLs: 
   Creation Date: 2015-08-14
   Depends: esx-base >= 5.5, esx-base <= 6.0.0, nsx-api <= 1, vmkapi_2_2_0_0
   Conflicts: 
   Replaces: 
   Provides: vsip = 1.0.0-0
   Maintenance Mode Required: False
   Hardware Platforms Required: 
   Live Install Allowed: True
   Live Remove Allowed: False
   Stateless Ready: True
   Overlay: True
   Tags: 
   Payloads: esx-vsip
~ #
```

As I was running NSX-v 6.2.1 on a ESXi 5.5 host, only 2 VIBs are installed.

To remove the VIBs, issue the following command:

```
esxcli software vib remove -n <VIB name>
```

```
~ # esxcli software vib remove -n esx-vxlan
Removal Result
   Message: The update completed successfully, but the system needs to be rebooted for the changes to be effective.
   Reboot Required: true
   VIBs Installed: 
   VIBs Removed: VMware_bootbank_esx-vxlan_5.5.0-0.0.2983935
   VIBs Skipped: 
~ # esxcli software vib remove -n esx-vsip
Removal Result
   Message: The update completed successfully, but the system needs to be rebooted for the changes to be effective.
   Reboot Required: true
   VIBs Installed: 
   VIBs Removed: VMware_bootbank_esx-vsip_5.5.0-0.0.2983935
   VIBs Skipped: 
~ #
```

Now that the VIBs have been removed, the host can be placed into Maintenance Mode and rebooted.

```
~ # esxcli system maintenanceMode set --enable true
~ # reboot
```

## **Update: 22/2/2016**

Just putting a couple of little lines here that I often use on customer environments.

NSX 6.1.X

```
esxcli software vib list | grep -i 'name\|\-\-\|vsip\|vxlan\|dvfilter\-switch'
```

NSX 6.2.X

```
esxcli software vib list | grep -i 'name\|\-\-\|vsip\|vxlan'
```
