---
title: "NSX-T 3.0 - Notification Watcher now Supported"
date: 2020-07-14

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
---

As part of the NSX-T 3.0 release, an API only feature that I had discussed [in a previous post](/2020/03/02/how-to-sync-a-dynamic-nsx-t-group-to-an-external-system/), now looks like it has had the experimental flag removed, which means it is now a supported API.

This is great news for those who were looking at this set of APIs and using it to prove that the dynamic group membership updates could now be sent to external systems for consumption.

More information about the APIs can be found in the NSX-T 3.0 API docs at the following link

[https://vdc-download.vmware.com/vmwb-repository/dcr-public/9b2ffac9-56c4-4587-b55f-bbd8ac78072a/780e6c8b-8fd8-4eaa-bc87-e00e98089a9b/api\_includes/system\_administration\_monitoring\_notifications.html](https://vdc-download.vmware.com/vmwb-repository/dcr-public/9b2ffac9-56c4-4587-b55f-bbd8ac78072a/780e6c8b-8fd8-4eaa-bc87-e00e98089a9b/api_includes/system_administration_monitoring_notifications.html)

If your browsing the API doc, its under section **3.7.3.7 - Notifications**

And for a practical proof of concept, make sure you go back and read the article linked above which shows how to get the dynamic group updates and push them into a Cisco ASA firewall using Ansible.
