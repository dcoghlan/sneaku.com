---
title: "vRealize Network Insight - Tips and Tricks"
draft: true
---

## Queries

Find all IP Sets that have not been statically included to a NSX Security Group.

```
IPset where Direct Parent Security Group is not set
```

Find all security groups which have the IP Set called "google\_dns\_1" statically included

```
security group where All Direct Child Group = 'google_dns_1'
```
