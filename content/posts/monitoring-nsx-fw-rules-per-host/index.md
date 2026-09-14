---
title: "Monitoring NSX FW Rules per host"
date: 2020-11-18
categories: 
  - "nsx"
  - "scripts"
  - "vmware"
Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
---

Today I was asked for some help in figuring out the number of NSX-T Distributed Firewall rules configured for all filters on a given ESXi host.

Why do we need to figure out this value? If you take a look at the configurations maximums page at [configmax.vmware.com](https://configmax.vmware.com), you'll see one of the NSX-T Configuration Maximums is the number of **Distributed Firewall Rules per Hypervisor Host**.

Up until recently (NSX-T 3.0.2), this number had always remained the same at 10,000 and was something that you should monitor to make sure you remained within this (soft) limit.

As you can see, this limit has been increased in the latest NSX-T 3.1.0 release:

```
--------------------------------------------------------------------------------
Firewall        : Distributed Firewall
--------------------------------------------------------------------------------
[MOD] [Distributed Firewall] Rules per Hypervisor Host; oldValue:10,000; newValue:120,000
```

Anyways, I went off on a bit of a tangent there.....

So, back to the original ask, my go-to option was to recommend a Powershell script written by Steve Ottavi, which uses SSH to connect to the host(s) and figure all this out for you ([read about it here](https://www.sneaku.com/2020/05/27/how-to-count-dfw-rules-per-esxi-host/)), however, there was a constraint:

> _ssh isn’t an option so needs to be shell.  Idea is to take the number generated & feed into vrops for trending.  Vrops can target the script & pull out the number as a metric._

I was given a starting piece of code that they had created so far, which was 90% of the way there.

All I needed to do was modify it to count the rules from not just DFW filters (ending in sfw.2), but also any other filters with applicable rules configured (potentially ending in .2 all the way up to .15). Then it needed to be able to save the total number of rule from all the filters in a variable so it can be sent somewhere.

In the end, this is what I came up with. I just displayed the count on the screen, but its possible to do whatever you want with the total count.

```
for n in $(summarize-dvfilter | grep -E '(^\s+name:\s+.*\.([2,4-9]|1[0-5]))$' | awk '{print $2}' ); do RULECOUNT=$(vsipioctl getrules -f $n | grep -i '\sat\s' | wc -l); COUNT=$(($COUNT + $RULECOUNT)); done
echo $COUNT
```

Now they will be able to use this in a script which can run locally on the ESXi host at a scheduled interval and send the results to vROPs (or even Log Insight) to be used for tracking/trending & alerting.

To see what else is possible when trying to log/send metrics from the host in this method of using a local shell script, have a read of one of my previous posts about [Monitoring DFW Heap Usage](https://www.sneaku.com/2017/06/06/monitoring-dfw-heap-usage/)
