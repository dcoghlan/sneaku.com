---
title: "Broadcom bnx2x driver and VXLAN offload"
date: 2015-05-25
categories: 
  - "nsx"
  - "vmware"
coverImage: "broadcom_240x240.jpg"
---

The Broadcom bnx2x NIC driver for VMware ESXi when installed on a ESXi 5.5 host is often an overlooked component when working with VMware NSX.

If you have one of the Broadcom NICs which supports VXLAN offload and uses the bnx2x driver, it is important to choose the correct driver version. Not all versions are created equal!

If you look through the release notes provided with all versions of the bnx2x driver download on the VMware website, it will show you a full history of changes and enhancements made to the driver.

An important change was made in version 1.78.56v\[50,55\].3 (Oct. 1, 2013)

```
1.78.56v[50,55].3 (Oct. 1, 2013)
===============================

Enhancements:
-------------
 1. Change:     Disabled vxlan offload by default since it is un-certified.

    Impact:     ESX5.5 only.
```

And then it looks like Broadcom had the VXLAN offloading certified for ESX5.5 and the following change was made in version 2.710.70v\[50,55\].3 (Oct. 7, 2014)

```
Version 2.710.70.v[50,55].3 (Oct. 7, 2014)
===============================
Internal FW 7.10.51

Enhancements:
-------------
 1. Requst :  Enabled 'enable_vxlan_ofld' module parameter for Vxlan offloads by default,
                previously it was disabled by default.
```

So this means that if you have the bnx2x driver 1.78.56v\[50,55\].3 or newer, but older than 2.710.70.v\[50,55\].3, VXLAN offloading in ESXi 5.5 is disabled in the driver by default due to it not being certified.

I have seen customer environments where the bnx2x drivers installed on the ESXi 5.5 hosts falls between the 2 versions where VXLAN was disabled by default which has caused some unexpected VXLAN performance results.

So if you find that your driver version falls within the range where VXLAN offloading was disabled by default and you are looking at implementing VMware NSX, you will need to ensure you have a driver version which has VXLAN offloading certified & enabled.

**Update - 1st June 2015:** To check to see if VXLAN offload has been disabled by default in any driver version, you can use the following command, which should show you what the default is.

```
~ # vmkload_mod -s bnx2x | grep -i -A 1 enable_vxlan_ofld
```

```
  enable_vxlan_ofld: int
    Allow vxlan TSO/CSO offload support.[Default is enabled, 1: enable vxlan offload, 0: disable vxlan offload]

```

If updating the driver is not an option due to other reasons, you can also manually enable VXLAN offloads. First you need to see what options (if any) are already set.

```
~ # esxcfg-module -g bnx2x
bnx2x enabled = 1 options = ''
```

And as you can see, there are no options set in the output above. So you can enabled it as follows:

```
~ # esxcfg-module -s 'enable_vxlan_ofld=1' bnx2x

~ # esxcfg-module -g bnx2x
bnx2x enabled = 1 options = 'enable_vxlan_ofld=1'
```

But what happens if there is another option already set, like RSS.

```
~ # esxcfg-module -g bnx2x
bnx2x enabled = 1 options = 'RSS=4'
```

The command would be as follows:

```
~ # esxcfg-module -s 'RSS=4 enable_vxlan_ofld=1' bnx2x
```

You will need to reboot the host for the changes to take affect, as apparently the driver cannot be unloaded whilst in use to apply the settings.

Once the host has rebooted, verify the setting has "stuck"

```
~ # esxcfg-module -g bnx2x
bnx2x enabled = 1 options = 'RSS=4 enable_vxlan_ofld=1'
```
