---
title: "Central CLI Update #1 for NSX 6.2.3/6.2.4"
date: 2016-09-17
categories: 
  - "nsx"
  - "vmware"
tags:
  - "central cli"

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
---

Following on from my previous post in regards to getting the [Central CLI API](http://www.sneaku.com/2016/09/06/central-cli-api-update-for-nsx-6-2-36-2-4/) working after an upgrade to 6.2.3/6.2.4, it seems that a nice little enhancement has been made which should help verifying and troubleshooting DFW data plane issues.

When looking at DFW filters and you wanted to view the contents of an address set, on the data plane you would issue the following command:

```
vsipioctl getaddrsets -f filter-name
```

And it would display the **complete** list of address sets for the particular filter.

> On some filters, this is acceptable, but on others it can be quite large. One particular environment I have been working on recently, would return roughly 260,000 lines of output when looking at the address sets.

So if you wanted to query a specific address set, you could use the -a option:

```
vsipioctl getaddrsets -f filter-name -a address_set
```

And it would display just the details of the address set specified.

When the Central CLI was introduced, the command to view the address set attached to a filter was as follows:

```
show dfw host host-id filter filter-name addrsets
```

And just like the command vsipioctl getaddrsets -f filter-name  it will display the complete address set for the particular filter.

But there lacked an equivalent option for the \-a addrset-name option.

With NSX vSphere 6.2.3/6.2.4 you can now issue the following command and query a specific address set on a filter.

```
show dfw host host-id filter filter-name addrsets addrset addrset-name
```

These are now the options you get when querying filter address sets:

```
nsx-m-01a> show dfw host host-52 filter nic-783592-eth0-vmware-sfw.2 address ?
  <cr>
  addrset  Show particular addrset information

```

```
nsx-m-01a> show dfw host host-52 filter nic-783592-eth0-vmware-sfw.2 address addrset ?
  ADDRSET  Show particular addrset information
```

And here it is in action:

```
nsx-m-01a> show dfw host host-52 filter nic-783592-eth0-vmware-sfw.2 addrsets addrset dst1011
addrset dst1011 {
ip 10.2.4.32,
ip 10.16.128.32,
}

```

I've also confirmed it works via the Central CLI API too.

I must be a bit of a geek, as this made me very happy!
