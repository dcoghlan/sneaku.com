---
title: "NSXv - DFW Log Format Change"
date: 2016-09-17
categories: 
  - "nsx"
  - "vmware"

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
---

In NSXv 6.2.2 and earlier, the format of the DFW logs has remained relatively the same for quite some time now. The following is a specific sample of the dfwpktlogs.log file from an NSXv 6.2.2 host.

```
[root@host-192-168-111-11:~] tail -f /var/log/dfwpktlogs.log
2016-09-17T09:41:50.979Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T09:41:50.979Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T09:41:50.979Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T09:41:51.026Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T09:41:51.026Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T09:41:51.026Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T09:41:51.026Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T09:41:51.089Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T09:41:51.135Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T09:41:51.792Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T09:41:51.792Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T09:41:51.792Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T09:41:52.897Z INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
```

The interesting thing about the sample logs shown above is that these are from 2 individual VMs, connected to 2 separated logical switches, but the VMs have the same IP address. This is a situation which can manifest itself when using NSX in either a multi-tenant environment or when using NSX to clone existing topologies.

As you can see, if you wanted to track down which of the VMs on this particular host the packets are coming from, its going to be a bit difficult.

In NSXv 6.2.3/6.2.4, the DFW log file format has been update with an extra piece of information.

```
2016-09-17T10:34:55.004Z 7295 INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T10:34:55.036Z 3603 INET match DROP domain-c46/1005 OUT 96 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T10:34:55.051Z 7295 INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T10:34:55.333Z 3603 INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T10:34:55.758Z 7295 INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T10:34:55.786Z 3603 INET match DROP domain-c46/1005 OUT 96 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T10:34:55.801Z 7295 INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T10:34:56.114Z 3603 INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
2016-09-17T10:34:56.451Z 3603 INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137
```

Directly after the timestamp of the log, there is a number. I've highlighted it in the log entries below:

2016-09-17T10:34:55.004Z **7295** INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137 2016-09-17T10:34:56.451Z **3603** INET match DROP domain-c46/1005 OUT 78 UDP 10.2.5.10/137->10.2.5.255/137

This number can be used to trace the log back to a particular vNic, and in turn, back to a particular VM.

To first step is to issue the vsipioctl getfilters  command. The identifying number in the logs matches the Filter Hash entry from the output of the command:

```
[root@host-192-168-111-12:~] vsipioctl getfilters

Filter Name              : nic-1744435-eth0-vmware-sfw.2
VM UUID                  : 50 01 41 b3 c5 25 95 57-f9 6b 59 43 fa 55 04 e1
VNIC Index               : 0
Service Profile          : --NOT SET--
Filter Hash              : 7295

Filter Name              : nic-1744441-eth0-vmware-sfw.2
VM UUID                  : 50 01 3d 6e 78 a0 e7 23-4a d0 56 a7 76 55 a4 59
VNIC Index               : 0
Service Profile          : --NOT SET--
Filter Hash              : 3603

```

If you have a lot of filters on the ESXi host, you can use the following command to return just the results your interested in:

```
vsipioctl getfilters | grep 3603 -B 5
```

```
[root@host-192-168-111-12:~] vsipioctl getfilters | grep 3603 -B 5

Filter Name              : nic-1744441-eth0-vmware-sfw.2
VM UUID                  : 50 01 3d 6e 78 a0 e7 23-4a d0 56 a7 76 55 a4 59
VNIC Index               : 0
Service Profile          : --NOT SET--
Filter Hash              : 3603

```

Now take the filter name and search for it in the output of the summarize-dvfilter  command:

```
[root@host-192-168-111-12:~] summarize-dvfilter
Fastpaths:
agent: dvfilter-faulter, refCount: 1, rev: 0x1010000, apiRev: 0x1010000, module: dvfilter
agent: ESXi-Firewall, refCount: 3, rev: 0x1010000, apiRev: 0x1010000, module: esxfw
agent: dvfilter-generic-vmware, refCount: 1, rev: 0x1010000, apiRev: 0x1010000, module: dvfilter-generic-fastpath
agent: dvfg-igmp, refCount: 1, rev: 0x1010000, apiRev: 0x1010000, module: dvfg-igmp
agent: dvfilter-generic-vmware-swsec, refCount: 3, rev: 0x1010000, apiRev: 0x1010000, module: dvfilter-switch-security
agent: bridgelearningfilter, refCount: 1, rev: 0x1010000, apiRev: 0x1010000, module: vdrb
agent: vmware-sfw, refCount: 3, rev: 0x1010000, apiRev: 0x1010000, module: vsip

Slowpaths:

Filters:
world 0 <no world>
 port 33554436 vmk0
  vNic slot 0
   name: nic-0-eth4294967295-ESXi-Firewall.0
   agentName: ESXi-Firewall
   state: IOChain Attached
   vmState: Detached
   failurePolicy: failOpen
   slowPathID: none
   filter source: Invalid
 port 50331654 vmk1
  vNic slot 0
   name: nic-0-eth4294967295-ESXi-Firewall.0
   agentName: ESXi-Firewall
   state: IOChain Attached
   vmState: Detached
   failurePolicy: failOpen
   slowPathID: none
   filter source: Invalid
world 1744435 vmm0:Win-02 vcUuid:'50 01 41 b3 c5 25 95 57-f9 6b 59 43 fa 55 04 e1'
 port 50331659 Win-02.eth0
  vNic slot 2
   name: nic-1744435-eth0-vmware-sfw.2
   agentName: vmware-sfw
   state: IOChain Attached
   vmState: Detached
   failurePolicy: failClosed
   slowPathID: none
   filter source: Dynamic Filter Creation
  vNic slot 1
   name: nic-1744435-eth0-dvfilter-generic-vmware-swsec.1
   agentName: dvfilter-generic-vmware-swsec
   state: IOChain Attached
   vmState: Detached
   failurePolicy: failClosed
   slowPathID: none
   filter source: Alternate Opaque Channel
world 1744441 vmm0:Win-01 vcUuid:'50 01 3d 6e 78 a0 e7 23-4a d0 56 a7 76 55 a4 59'
 port 50331660 Win-01.eth0
  vNic slot 2
   name: nic-1744441-eth0-vmware-sfw.2
   agentName: vmware-sfw
   state: IOChain Attached
   vmState: Detached
   failurePolicy: failClosed
   slowPathID: none
   filter source: Dynamic Filter Creation
  vNic slot 1
   name: nic-1744441-eth0-dvfilter-generic-vmware-swsec.1
   agentName: dvfilter-generic-vmware-swsec
   state: IOChain Attached
   vmState: Detached
   failurePolicy: failClosed
   slowPathID: none
   filter source: Alternate Opaque Channel
[root@host-192-168-111-12:~]
```

As you can see, its now possible to be able to track the particular log entry back to a VM and narrow it down to an individual vNIC.

Again, if you have a lot of VMs running on the ESXi host, you can use the following command to return just the results your interested in:

```
summarize-dvfilter | grep 'port\|sfw\.2' | grep nic-1744441-eth0-vmware-sfw.2 -B 1
```

```
[root@host-192-168-111-12:~] summarize-dvfilter | grep 'port\|sfw\.2' | grep nic-1744441-eth0-vmware-sfw.2 -B 1
port 50331660 Win-01.eth0
name: nic-1744441-eth0-vmware-sfw.2

```

Overall I think this is a welcomed update to the DFW logging format, however I have seen that the change has caused a few syslog servers to either ignore the logs as the format has changed, or not display them correctly as the expressions used to parse the logs didn't cater for the new field. This is something to keep in mind when it comes to looking at upgrading from an earlier version to 6.2.3 or above.
