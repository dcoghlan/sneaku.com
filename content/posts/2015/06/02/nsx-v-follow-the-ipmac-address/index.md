---
title: "NSX-v: Follow the IP/MAC address"
date: 2015-06-02
categories: 
  - "nsx"
  - "vmware"

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
---

Recently on a customer site we had a peculiar scenario where we were deploying a VM into a NSX environment via vRA and the operation was failing due to an unknown reason. However, we noticed that the IP address that vRA had allocated for the new VM was still responding to our pings even though the provisioning process had failed and the VM was never actually deployed...... so what was responding to our pings and where was it????

We tried accessing it via telnet, ssh, RDP, VNC, SNMP and a few other methods to figure out what it was but had no luck (we were not the only ones working in the environment so there were machines deployed we knew nothing about). We knew it must be a VM as the particular subnet was associated with a VXLAN, and we could see within the vSphere Web Client all the VMs that were connected to the logical switch in question, but due to the fact the VM template used to build these test machines didn't have VMware Tools installed, we couldn't see the IP addresses on the VMs in the vSphere Web Client.

So the question asked was "how do you find out what VM an IP address belongs to, in an NSX-v environment?

# Cisco Commands

In a traditional physical network, to figure out where an IP address is configured, I would normally jump on the gateway for the L3 subnet, and look at the ARP tables of the L3 device. On a Cisco device, the command I would use would be something along the lines of:

```
Router#show ip arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.29.4.1               0   0050.56c0.0002  ARPA   GigabitEthernet1
Internet  10.29.4.222             0   8c70.5a12.8454  ARPA   GigabitEthernet1
Internet  10.29.4.101             - 000c.292f.e634  ARPA   GigabitEthernet1
Internet  10.29.4.254            13   0050.56f8.b138  ARPA   GigabitEthernet1
Internet  10.29.32.254            2   0254.003c.fa14  ARPA   GigabitEthernet1
```

This would give me the MAC address associated to the IP address in question, and also the physical interface for which the router has associated the information to.

Next I would jump onto the L2 switch connected to the interface which the router has associated the ARP information to, and look at the mac-address tables to figure out which port the applicable MAC address is associated to.

```
switch1#show mac address-table address 8c70.5a12.8454
          Mac Address Table
-------------------------------------------
  
Vlan    Mac Address       Type        Ports
---- ----------- -------- -----
  10    8c70.5a12.8454    DYNAMIC     Gi0/25
Total Mac Addresses for this criterion: 1
switch1#
```

Now you know the physical port the phantom device is connected to, and if your switch port descriptions are kept up to date, it should tell you what it is connected to the port. If it connects to another switch, you repeat the process until you find the end device and if required, trace the physical cable. If it went to a blade enclosure or a hypervisor, I would then go and bug the team who looked after those environments so they could tell me what the device is ;)

But what if there are no physical switch ports or cables like in a NSX-v environment and the IP in question is a VM connected to a logical switch?

# NSX-v Commands

First, we need to identify the master controller for the vni.

SSH into one of the controllers, or jump on the console of one, and run the following command:

```
nsx-controller # show control-cluster logical-switches vni 5001
VNI      Controller      BUM-Replication ARP-Proxy Connections VTEPs
5001     10.29.90.61     Enabled         Enabled   3           3

```

We can see the master controller for vni _5001_ is _10.29.90.61_.

Now jump onto the controller _10.29.90.61_ and we will look at the ARP tables associated with vni 5001. We can issue the following command on the master controller:

_(the IP address we are going to follow in this example is 10.29.84.141)_

```
nsx-controller # show control-cluster logical-switches arp-table 5001
VNI      IP              MAC               Connection-ID
5001     10.29.84.141   00:50:56:be:81:d3 10183
5001     10.29.84.142   00:50:56:be:0d:08 10187

```

Now we have the MAC address _(00:50:56:be:81:d3)_ which belongs to the IP address 10.29.84.141, along with the Connection-ID _(10183)_ of the ESXi host.

_**OPTIONAL**_: _If we issue the following command we can see which VTEP the MAC address is mapped to._

```
nsx-controller # show control-cluster logical-switches mac-table 5001
VNI      MAC               VTEP-IP         Connection-ID
5001     00:50:56:be:81:d3 10.29.97.1      10183
5001     00:50:56:be:0d:08 10.29.97.2      10187
5001     00:50:56:be:dd:46 10.29.97.2      10187
5001     00:50:56:be:34:de 10.29.97.3      10188

```

You can see from both the commands above that IP address _10.29.84.141_ has a MAC address _00:50:56:be:81:d3,_ is mapped to VTEP _10.29.97.1,_ which has a Connection-ID of _10183._

Next we need a way to figure out which host the Connection-ID belongs to.

Thanks to Dmitri Kalintsev and [this post,](https://telecomoccasionally.wordpress.com/2014/12/25/nsx-for-vsphere-controller-connections-and-vteps/) we know that we can use the following command to display the connection-table, which will show the ESXi management host IP address associated to the Connection-ID:

```
nsx-controller # show control-cluster logical-switches connection-table 5001
Host-IP         Port  ID
10.29.91.3      12836 10183
10.29.91.4      50722 10187
10.29.91.2      57147 10188

```

We can see that Connection-ID _10183_ belongs to Host-IP _10.29.91.3_.

Now we can SSH into ESXi host _10.29.91.3_ and run the following command to look at what VM and vNIC the MAC address belongs to:

```
~ # net-stats -l
PortNum          Type SubType SwitchName       MACAddress         ClientName
67108866            4       0 vSwitch1         74:e6:e2:ba:f9:78  vmnic3
67108869            3       0 vSwitch1         00:50:56:63:d2:53  vmk2
83886082            3       0 vSwitch2         00:50:56:6f:f8:e8  vmk1
83886083            4       0 vSwitch2         74:e6:e2:ba:f9:75  vmnic2
100663298           4       0 DvsPortset-1     74:e6:e2:ba:f8:a4  vmnic1
100663300           3       0 DvsPortset-1     00:50:56:6c:5a:c2  vmk4
100663301           3       0 DvsPortset-1     00:50:56:67:f8:c7  vmk3
100663302           3       0 DvsPortset-1     74:e6:e2:ba:f8:a1  vmk0
100663306           4       0 DvsPortset-1     74:e6:e2:ba:f8:a1  vmnic0
100663310           3       0 DvsPortset-1     00:50:56:6a:06:6a  vmk5
100663353           5       9 DvsPortset-1     00:50:56:be:81:d3  sneaku-scan01.eth1
100663354           5       9 DvsPortset-1     00:50:56:be:de:c0  sneaku-scan02.eth0
100663356           5       7 DvsPortset-1     00:50:56:be:ad:f8  NSX_Controller_4e6a2ed1-f438-4791-a8ae-07bc782b3735.eth0
100663359           5       9 DvsPortset-1     00:50:56:be:4d:12  NSX Manager.eth0
```

And just in case you were wondering, the IP Address in question belonged to a machine which was manually provisioned into the wrong subnet/logical switch as part of some previous testing, but was forgotten about. We ended up deleting the machine :)

Hope this helps.
