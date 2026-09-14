---
title: "NSX-v: The Penny Drops"
date: 2016-01-26
categories: 
  - "nsx"
  - "vmware"
tags: 
  - "nsx"
  - "vmware"
coverImage: "lightbuld.png"
---

I was onsite implementing NSX-v for a customer, and part of the installation in this environment requires that we allocate the VTEP pnics to the VXLAN transport VLAN. To do this I had to liaise with the Network Operations guy. After laying it out for him that although these VTEP interfaces are going to be used for "VM Data" (his words, not mine) they will only need to be an access port in VLAN xxx, the penny finally dropped for this guy on why they are going the NSX-v route.

You see in this environment, there are 36 ESXi hosts, (1 x 32 host cluster for compute and a 4 host cluster for NSX Mgmt & Edge devices), all running in 4 blade chassis. So not only do you have to check the network connectivity of the blade to the internal chassis switches, you must also make sure the upstream switch connecting the internal chassis switch to the outside world is also configured correctly. We were implementing a dual VTEP design, so each ESXi host had 2 NICs assigned as dvUplinks that connected to the 2 internal chassis switches. Each chassis switch then had 4 uplinks to the 2 x physical switches.

> In total, to allocate a new VLAN in this environment would involve configuring and checking a minimum of 104 ports!! And that's just counting the "connections" which could comprise of checking/configuring a device on each end of the connection.

When I asked him could he make sure the pnics to be used for the VTEPs on each blade were in VLAN xxx, he said it would take him at least an hour or two (to check just 8 of the hosts initially), to go through all of the configs etc just to see if was already allowed, and then if things needed to be changed, it could take longer.

This was just a simple task of connecting NICs to a single VLAN. Now just imagine that this company wanted to implement a new VLAN and present it to the "cloud" environment. This poor Network Operations guy, would literally spend hours having to configure this VLAN on multiple network switches and blade chassis etc. This isn't very agile.

Now with NSX, once its all up and running, if they need to create a new Logical Switch (VLAN) for their "cloud" environment, its as simple as creating the logical switch with a couple of clicks, and wham bam thankyou mam, the logical switch is available within your specified transport zone. If you need it routable, just connect it to your Edge Gateway or Distributed Logical Router and away you go. With NSX this can all be done programmatically now too!

After I went back to my work area to continue the NSX installation, I did overhear the following:

_**Network Ops**: Oh wow, does this mean that we won't have to do these VLAN changes every time to the cloud environment? That's gonna cut down our workload heaps._

_**Network Architect**: That's one of the reasons we are doing this!!!!!_
