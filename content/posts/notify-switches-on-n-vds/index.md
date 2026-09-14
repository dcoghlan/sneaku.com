---
title: "Notify Switches on N-VDS"
date: 2020-06-01
categories: 
  - "nsx"
  - "vmware"
---

Recevied an interesting question today about whether a N-VDS supports the "Notify Switches" feature that is available in regular VDS. And as I didn't know the answer to this off the top of my head, it was time to do some digging.

First we needed an environment where I had a N-VDS configured, and it just so happened I had a 2.5.1 setup running.

Jumping onto a host, first thing was to find what the N-VDS was actually called.

```
[root@host-192-168-109-153:~] esxcfg-vswitch -l
Switch Name      Num Ports   Used Ports  Configured Ports  MTU     Uplinks
vSwitch0         2560        1           128               1500
  PortGroup Name        VLAN ID  Used Ports  Uplinks
  VM Network            0        0
Switch Name      Num Ports   Used Ports  Uplinks
NVDS-Overlay     2560        12          vmnic1,vmnic0
```

So my N-VDS has been named "NVDS-Overlay". Now I can use that to take a look at some of its configuration.

> The output below is only showing the global properties section of the output.

```
[root@host-192-168-109-153:~] net-dvs -l NVDS-Overlay
switch 03 f6 d6 82 99 fb 49 0a-b4 0f 70 21 fd 22 7b ef (vswitch)
        max ports: 2560
        global properties:
                com.vmware.common.opaqueDvs = true , propType = CONFIG
                com.vmware.common.alias = NVDS-Overlay , propType = CONFIG
                com.vmware.common.portset.mtu = 1600 , propType = CONFIG
                com.vmware.etherswitch.cdp = LLDP, listen
                        propType = CONFIG
                com.vmware.common.respools.version = version3 ,  propType = CONFIG
                com.vmware.common.respools.cfg:
                        netsched.pools.persist.ft:0:50:-1:255
                        netsched.pools.persist.hbr:0:50:-1:255
                        netsched.pools.persist.iscsi:0:50:-1:255
                        netsched.pools.persist.mgmt:0:50:-1:255
                        netsched.pools.persist.nfs:0:50:-1:255
                        netsched.pools.persist.vdp:0:50:-1:255
                        netsched.pools.persist.vm:0:100:-1:255
                        netsched.pools.persist.vmotion:0:50:-1:255
                        netsched.pools.persist.vsan:0:50:-1:255
                        propType = CONFIG
                com.vmware.common.respools.sched:
                        active
                        propType = CONFIG
                com.vmware.common.version = 0x64. 0. 0. 0
                        propType = CONFIG
                com.vmware.vswitch.teaming:
                        load balancing = first uplink (i.e. explicit)
                        link selection = link state up;
                        link behavior = notify switch; best effort on failure; shotgun on failure;  << -- This is where it is set
                        active = uplink1,
                        standby = uplink2,
                        propType = CONFIG POLICY
                com.vmware.extraconfig.opaqueDvs.ens = false ,  propType = CONFIG
                com.vmware.extraconfig.opaqueDvs.pnicZone = ad3282ab-4c8b-4feb-b9bb-696a7c8a8bc9,9287f556-b28d-4c66-a009-90d8af0d9cd6 ,  propType = CONFIG
                com.vmware.extraconfig.opaqueDvs.status = up ,  propType = CONFIG
                com.vmware.common.uplinkPorts:
                        uplink1, uplink2
                        propType = CONFIG
```

And we can see from the output that "notify switch" appears to be set

```
link behavior = notify switch; best effort on failure; shotgun on failure;
```

Another method to checking this is to use nsxdp-cli

```
[root@host-192-168-109-153:~] nsxdp-cli vswitch teaming policy get -dvs NVDS-Overlay
loadBalance: FO_EXPLICIT
linkCriteria: DRV_PRESENT LINK_STATE
percentError: 0
fullDuplex: True
speed: 10
flags: SHOTGUN BEST_EFFORT NOTIFY_SWITCH
activeUplinkPorts: uplink1
standbyUplinkPorts: uplink2
```

So as you can see, it appears to be enabled by default when using N-VDS.
