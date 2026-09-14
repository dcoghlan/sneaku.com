---
title: "Cisco ASA Logging"
date: 2015-05-25
coverImage: "Cisco_240x240.jpg"
---

When logging is enabled on a Cisco ASA, it often logs way to much information and makes it difficult to troubleshoot when there are issues to be looked at.

Below is a config that can be pasted into an ASA which will disable most “noisy” logs and leave you with the denies, and most other relevant logs. This config will cut down your logging considerably.

```
asdm history enable
logging enable
logging facility 20
logging timestamp
logging emblem
logging standby
logging console critical
logging monitor debugging
logging buffered informational
logging trap informational
logging asdm informational
logging history alerts
 
asdm history enable

! Build outbound TCP connection is not logged
no logging message 302013
 
! Teardown outbound TCP connection is not logged
no logging message 302014
 
! Build outbound UDP connection is not logged
no logging message 302015
 
! Teardown outbound UDP connection is not logged
no logging message 302016
 
! Build outbound ICMP connection is not logged
no logging message 302020
 
! Teardown outbound ICMP connection is not logged
no logging message 302021
 
! User accessed url is not logged
no logging message 304001
 
! Build dynamic TCP xlate is not logged
no logging message 305011
 
! Teardown dynamic TCP xlate is not logged
no logging message 305012
 
! Build local-host is not logged
no logging message 609001
 
! Teardown local-host is not logged
no logging message 609002
 
! Constructing * hash payload is not logged
no logging message 715046
 
! Processing * hash payload is not logged
no logging message 715047
 
! IKE keepalive message is not logged
no logging message 715075
no logging message 715036
 
! IKE decode message is not logged
no logging message 713236
```
