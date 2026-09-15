---
title: "Scripting syslog server configurations on NSX-v Controllers"
date: 2015-05-28
categories: 
  - "nsx"
  - "scripts"
  - "vmware"

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
cover:
  image: "images/codeimage.png"
---

Its well documented that the only way to configure syslog settings on NSX-v controllers is via the REST API. One of the things I find myself constantly doing over and over again on customer engagements is configuring syslog servers details on the NSX-v controllers, whether its because I am at a new client, or someone decides to change the destination for their syslog and we need to update them all.

As this seems to be something I keep repeating, I have had it on my TODO list for quite some time to create a nice script which will do the hard work for me.

As usual, the script I have created is written in Python and the only external module required is the [Requests](http://docs.python-requests.org/en/latest/) module. All others should be in a standard installation of Python.

Here you can see the main help for the script. Each time the script is run you need to provide the NSX Manager IP/FQDN and optionally a username, if no username is specified it will use the default admin account on the NSX Manager.

```
python nsx-controller-syslog.py -h
```

```
usage: nsx-controller-syslog.py [-h] -nsxmgr nsxmgr [-user [user]]
                                {add,list,del} ...

List/Add/Delete NSX-v Controller syslog server configuration.

positional arguments:
  {add,list,del}
    add           Add syslog servers on all controllers
    list          List all controllers
    del           Delete all syslog servers on all controllers

optional arguments:
  -h, --help      show this help message and exit
  -nsxmgr nsxmgr  NSX Manager hostname, FQDN or IP address
  -user [user]    OPTIONAL - NSX Manager username (default: admin)
```

The script has 3 actions it can perform:

- **add** - This will add the syslog configuration to all the controllers in the system
- **list** - This will list the syslog configuration of all the controllers in the system
- **del** - This will delte the syslog configuration of all the controllers in the system

When running the list action, no further arguments are required.

```
python nsx-controller-syslog.py list -h
```

```
usage: nsx-controller-syslog.py list [-h]

optional arguments:
  -h, --help  show this help message and exit
```

When running the del action, no further arguments are required.

```
python nsx-controller-syslog.py del -h
```

```
usage: nsx-controller-syslog.py del [-h]

optional arguments:
  -h, --help  show this help message and exit
```

However when invoking the add action, it requires additional arguments as shown below

```
python nsx-controller-syslog.py add -h
```

```
usage: nsx-controller-syslog.py add [-h] -dest IP/FQDN [-protocol [protocol]]
                                    [-port [port]] [-level [Level]]

optional arguments:
  -h, --help            show this help message and exit
  -dest IP/FQDN         syslog server IP or FQDN
  -protocol [protocol]  OPTIONAL - UDP | TCP (default: UDP)
  -port [port]          OPTIONAL - Port number (default: 514)
  -level [Level]        OPTIONAL - Syslog Level (default: INFO)
```

The following are some examples showing how to run the script and also the output presented.

This one will list all the controller syslog configurations

```
python nsx-controller-syslog.py -nsxmgr 10.29.4.11 list
```

```
NSX Manager password:

#########################################################################################
                                    Current Settings                                     
#########################################################################################
Object ID       IP Address    Status   Syslog Server               Port  Protocol Level 
--------------- ------------- -------- --------------------------- ----- -------- ------
controller-1    10.29.4.41    RUNNING  splunk.sneaku.com           514   UDP      INFO  
controller-2    10.29.4.42    RUNNING  splunk.sneaku.com           514   UDP      INFO  
controller-3    10.29.4.43    RUNNING  splunk.sneaku.com           514   UDP      INFO
```

When using the add actions, if you just specify the destination to send the syslog data to, it will use the default of UDP/514 and set the logging level to INFO. It will show you the details both before and after the new configuration occurs.

```
python nsx-controller-syslog.py -nsxmgr 10.29.4.11 add -dest splunk.sneaku.com
```

```
NSX Manager password:

#########################################################################################
                                      Old Settings                                       
#########################################################################################
Object ID       IP Address    Status   Syslog Server               Port  Protocol Level 
--------------- ------------- -------- --------------------------- ----- -------- ------
controller-1    10.29.4.41    RUNNING                                                   
controller-2    10.29.4.42    RUNNING                                                   
controller-3    10.29.4.43    RUNNING                                                 

SUCCESS: Configured syslog server splunk.sneaku.com on controller-1
SUCCESS: Configured syslog server splunk.sneaku.com on controller-2
SUCCESS: Configured syslog server splunk.sneaku.com on controller-3

#########################################################################################
                                      New Settings                                       
#########################################################################################
Object ID       IP Address    Status   Syslog Server               Port  Protocol Level 
--------------- ------------- -------- --------------------------- ----- -------- ------
controller-1    10.29.4.41    RUNNING  splunk.sneaku.com           514   UDP      INFO  
controller-2    10.29.4.42    RUNNING  splunk.sneaku.com           514   UDP      INFO  
controller-3    10.29.4.43    RUNNING  splunk.sneaku.com           514   UDP      INFO
```

Or if you prefer, you can also specify the port, protocol and logging level if required. It will show you the details both before and after the new configuration occurs.

```
python nsx-controller-syslog.py -nsxmgr 10.29.4.11 add -dest splunk.sneaku.com -protocol TCP -port 5514 -level INFO

```

```
NSX Manager password:

#########################################################################################
                                      Old Settings                                       
#########################################################################################
Object ID       IP Address    Status   Syslog Server               Port  Protocol Level 
--------------- ------------- -------- --------------------------- ----- -------- ------
controller-1    10.29.4.41    RUNNING                                                   
controller-2    10.29.4.42    RUNNING                                                   
controller-3    10.29.4.43    RUNNING                                                 

SUCCESS: Configured syslog server splunk.sneaku.com on controller-1
SUCCESS: Configured syslog server splunk.sneaku.com on controller-2
SUCCESS: Configured syslog server splunk.sneaku.com on controller-3

#########################################################################################
                                      New Settings                                       
#########################################################################################
Object ID       IP Address    Status   Syslog Server               Port  Protocol Level 
--------------- ------------- -------- --------------------------- ----- -------- ------
controller-1    10.29.4.41    RUNNING  splunk.sneaku.com           5514  TCP      INFO  
controller-2    10.29.4.42    RUNNING  splunk.sneaku.com           5514  TCP      INFO  
controller-3    10.29.4.43    RUNNING  splunk.sneaku.com           5514  TCP      INFO
```

If you try and configure a controller which already has syslog configuration on it, it will throw an error.

```
python nsx-controller-syslog.py -nsxmgr 10.29.4.11 add -dest loginsight.sneaku.com -protocol UDP -port 514 -level INFO

```

```
NSX Manager password:

#########################################################################################
                                      Old Settings                                       
#########################################################################################
Object ID       IP Address    Status   Syslog Server               Port  Protocol Level 
--------------- ------------- -------- --------------------------- ----- -------- ------
controller-1    10.29.4.41    RUNNING  splunk.sneaku.com           5514  TCP      INFO  
controller-2    10.29.4.42    RUNNING  splunk.sneaku.com           5514  TCP      INFO  
controller-3    10.29.4.43    RUNNING  splunk.sneaku.com           5514  TCP      INFO                                               

ERROR: Syslog servers are already set on controller-1. Delete settings before trying to add new ones.
ERROR: Syslog servers are already set on controller-2. Delete settings before trying to add new ones.
ERROR: Syslog servers are already set on controller-3. Delete settings before trying to add new ones.

#########################################################################################
                                      New Settings                                       
#########################################################################################
Object ID       IP Address    Status   Syslog Server               Port  Protocol Level 
--------------- ------------- -------- --------------------------- ----- -------- ------
controller-1    10.29.4.41    RUNNING  splunk.sneaku.com           5514  TCP      INFO  
controller-2    10.29.4.42    RUNNING  splunk.sneaku.com           5514  TCP      INFO  
controller-3    10.29.4.43    RUNNING  splunk.sneaku.com           5514  TCP      INFO
```

And lastly if you want to delete the current syslog configuration on all controllers so that you can add new ones.  It will show you the details both before and after the delete occurs.

```
python nsx-controller-syslog.py -nsxmgr 10.29.4.11 del
```

```
NSX Manager password:

#########################################################################################
                                      Old Settings                                       
#########################################################################################
Object ID       IP Address    Status   Syslog Server               Port  Protocol Level 
--------------- ------------- -------- --------------------------- ----- -------- ------
controller-1    10.29.4.41    RUNNING  splunk.sneaku.com           514   UDP      INFO  
controller-2    10.29.4.42    RUNNING  splunk.sneaku.com           514   UDP      INFO  
controller-3    10.29.4.43    RUNNING  splunk.sneaku.com           514   UDP      INFO 

SUCCESS: Deleted syslog server configuration on controller-1
SUCCESS: Deleted syslog server configuration on controller-2
SUCCESS: Deleted syslog server configuration on controller-3

#########################################################################################
                                      New Settings                                       
#########################################################################################
Object ID       IP Address    Status   Syslog Server               Port  Protocol Level 
--------------- ------------- -------- --------------------------- ----- -------- ------
controller-1    10.29.4.41    RUNNING                                                   
controller-2    10.29.4.42    RUNNING                                                   
controller-3    10.29.4.43    RUNNING
```

As usual, the script is located on my GitHub site [here](https://github.com/dcoghlan/NSX-Controller-Syslog)
