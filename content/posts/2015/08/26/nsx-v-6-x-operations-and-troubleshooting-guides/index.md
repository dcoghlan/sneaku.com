---
title: "NSX-v 6.x: Operations and Troubleshooting Guides"
date: 2015-08-26
categories: 
  - "nsx"
  - "vmware"

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
cover:
  image: "images/Screen-Shot-2015-08-27-at-7.20.17-am_430x1000.png"
  relative: true
  hiddenInList: false
  hiddenInSingle: true
---

Operating and troubleshooting a NSX-v environment can sometimes be a daunting task, especially if the customer had the environment setup by an external party (ie. VMware PSO or a VMware Partner). And so over the past few weeks, VMware have released 2 pieces of collateral which I am finding answer a lot of questions that I am normally asked by customers.

The first is the NSX-v Operations Guide, v6.1 ([https://communities.vmware.com/docs/DOC-30079](https://communities.vmware.com/docs/DOC-30079)) that is posted on the VMware Communities site. At a high level, this document touches on the following topics:

- Statistics Monitoring
- Flow Visibility (IPFIX)
- IPFIX Templates
- Packet Visibility
- System Events and Status (via API)
- Backup and Restore
- Individual Component Backups
- Failure and Recovery Scenarios
- Troubleshooting Backup and Restore Operations
- User Management
- How to use the NSX API
- Logging
- Log Message Codes

The second is the following Master KB article on [Understanding and troubleshooting VMware NSX for vSphere 6.x (2122691)](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2122691). This KB article should be the first port of call when troubleshooting NSX-v related issues, and will give you a head start on narrowing down or even resolving issues. From what I understand, this article is also referenced by GSS to help troubleshoot issues when you log a support ticket with VMware.

I highly recommend reading and bookmarking both of these documents.
