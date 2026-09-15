---
title: "How to count DFW rules per ESXi host."
date: 2020-05-27
categories: 
  - "api"
  - "nsx"
  - "vmware"

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
---

If you've heard me speak at VMworld on NSX Distributed Firewall best practises, you would have heard me speak about the importance of using the Applied To option when configuring DFW rules. One of the metrics using the Applied To option influences is the total number of rules configured per host.

> If you haven't seen the VMworld session, I've uploaded it to YouTube for easier viewing - [https://youtu.be/fX9pwiIeMps](https://youtu.be/fX9pwiIeMps)

As per the published configurations on [configmax.vmware.com](https://configmax.vmware.com/), the maximum number of rules supported per host is as follows:

<figure>

| **Product** | **Maximum** |
| --- | --- |
| NSX Data Center for vSphere | 10,000 |
| VMware NSX-T | 10,000 |

<figcaption>

Maximum number of rules per host

</figcaption>

</figure>

The number of rules per host is calculated by adding up all the rules configured on each filter on a host.

Whilst most deployments out there will utilise the filters on slot 2 (DFW), this "rules per host" figure also covers any Service Insertion (SI), Identity Firewall (IDFW) or Intrusions Detection (IDS) rules on the host. These other features utilise filters on the other "slots".

It is possible to calculate this figure manually, using vsipioctl commands for a spot check, but what if you need to keep an eye on this figure over time, or you have a large number of hosts, or if your like me and like to be able to automate these thing.

Well as it turns out, another VMware colleague, Steve Ottavi created a script which utilizes Powershell and Posh-SSH to SSH to the ESXi hosts and calculates the figures for you.

Steve has made the script available on [GitHub](https://github.com/CorvaxTC/powercli-nsx) and [code.vmware.com](https://code.vmware.com/samples/7254/nsx-get-rules-count-v-1-.-2?h=Sample#) so feel free to download it and use it in your environments.
