---
title: "NSX-v Quick Tip – ESG Enable Mode"
date: 2015-07-29

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
---

When using the CLI of an Edge Services Gateway there are some commands that can only be run from privileged mode. This is similar in fashion to the Privileged EXEC mode in Cisco IOS devices, and also the commands to turn on and off privileged mode are identical.

To turn on privileged mode on an ESG can be achieved with the following command:

```
enable
```

> When it prompts you for the password, you enter the Admin password that you logged onto the CLI with.

```
test-edge-01>

test-edge-01> enable

Password: 

test-edge-01#
```

To turn off or exit privileged mode without logging out of the CLI issue the following command:

```
disable
```

```
test-edge-01# disable

test-edge-01>
```
