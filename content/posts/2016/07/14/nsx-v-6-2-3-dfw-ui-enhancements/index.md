---
title: "NSX-v 6.2.3 - DFW UI Enhancements"
date: 2016-07-14

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
---

With the recent release of NSX vSphere 6.2.3 there were a couple of subtle updates to the Distributed Firewall UI that aren't obvious at first glance.

## Rule ID now shown by default.

The Rule ID is now shown in the Distributed Firewall UI by default. No need to continually enable the visibility of the column every time you log in.

## Adding a DFW rule doesn't scroll up to the top of the screen.

When working with large rule sets, when you wanted to add a new rule into a section, it would always return you to the top of the section/rule base upon insertion of the rule, thus forcing you to have to scroll again to try and find the blank rule that has now been inserted so that you can configure it before publishing it.

Whilst it hasn't been totally fixed and it will still scroll after adding/inserting a new rule, it will only scroll to a point where the newly added rule is at the top of the rules now.

## Cannot modify DFW sections created by Service Composer anymore.

Service Composer Sections can now no longer be modified in the UI. Previously (6.2.2) a user was able to manually add a rule to the top of a DFW section that was created/managed by Service Composer. This has now been disabled and should help in stopping any Service Composer policies going out of sync due to someone manually creating rules in the DFW Firewall UI within those sections.

The release notes are [here](http://pubs.vmware.com/Release_Notes/en/nsx/6.2.3/releasenotes_nsx_vsphere_623.html) for the official list of updates.
