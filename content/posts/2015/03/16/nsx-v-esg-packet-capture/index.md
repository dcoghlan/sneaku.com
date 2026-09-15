---
title: "NSX-v: ESG Packet Capture"
date: 2015-03-16
categories: 
  - "nsx"
  - "vmware"
tags: 
  - "nsx"
  - "vmware"

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
---

Whilst troubleshooting at a client today, I needed to perform a packet capture on one of the Edge Services Gateways in the environment. Performing a packet capture is often very helpful in diagnosing a range of different issues.

To kick off a packet capture you can jump on the console of the ESG or like I am doing in this example, open up an SSH session to the ESG.

You will need to know what interface to run the capture on, so run the following command to list out all the interfaces _(for ease of reading I have removed all the interfaces that were showing down/down from the output)_

```
vShield-edge-3-0> show interface
Interface VDR is up, line protocol is up
  index 2 metric 1 mtu 1500 <UP,BROADCAST,RUNNING,NOARP>
  HWaddr: 12:ee:19:2e:18:f6
  inet6 fe80::10ee:19ff:fe2e:18f6/64
  proxy_arp: disabled
  Auto-duplex (Full), Auto-speed (2157Mb/s)
    input packets 0, bytes 0, dropped 0, multicast packets 0
    input errors 0, length 0, overrun 0, CRC 0, frame 0, fifo 0, missed 0
    output packets 0, bytes 0, dropped 0
    output errors 0, aborted 0, carrier 0, fifo 0, heartbeat 0, window 0
    collisions 0

Interface br-sub is up, line protocol is up
  index 13 metric 1 mtu 1500 <UP,BROADCAST,RUNNING,MULTICAST>
  inet6 fe80::5890:b7ff:fecd:6c9c/64
  proxy_arp: disabled
  Auto-duplex (Full), Auto-speed (2157Mb/s)
    input packets 0, bytes 0, dropped 0, multicast packets 0
    input errors 0, length 0, overrun 0, CRC 0, frame 0, fifo 0, missed 0
    output packets 319, bytes 27498, dropped 0
    output errors 0, aborted 0, carrier 0, fifo 0, heartbeat 0, window 0
    collisions 0

Interface lo is up, line protocol is up
  index 1 metric 1 mtu 16436 <UP,LOOPBACK,RUNNING>
  inet 127.0.0.1/8
  inet6 ::1/128
  proxy_arp: disabled
  Auto-duplex (Full), Auto-speed (2157Mb/s)
    input packets 10738, bytes 1550427, dropped 0, multicast packets 0
    input errors 0, length 0, overrun 0, CRC 0, frame 0, fifo 0, missed 0
    output packets 10738, bytes 1550427, dropped 0
    output errors 0, aborted 0, carrier 0, fifo 0, heartbeat 0, window 0
    collisions 0

Interface vNic_0 is up, line protocol is up
  index 3 metric 1 mtu 1500 <UP,BROADCAST,RUNNING,MULTICAST>
  HWaddr: 00:50:56:9d:74:93
  inet6 fe80::250:56ff:fe9d:7493/64
  inet 10.29.254.241/24
  proxy_arp: disabled
  Auto-duplex (Full), Auto-speed (2157Mb/s)
    input packets 22451, bytes 4017743, dropped 1535, multicast packets 4540
    input errors 0, length 0, overrun 0, CRC 0, frame 0, fifo 0, missed 0
    output packets 44431, bytes 7692037, dropped 0
    output errors 0, aborted 0, carrier 0, fifo 0, heartbeat 0, window 0
    collisions 0

Interface vNic_1 is up, line protocol is up
  index 6 metric 1 mtu 1500 <UP,BROADCAST,RUNNING,MULTICAST>
  HWaddr: 00:50:56:9d:0e:30
  inet 10.29.2.241/28
  inet6 fe80::250:56ff:fe9d:e30/64
  proxy_arp: disabled
  Auto-duplex (Full), Auto-speed (2157Mb/s)
    input packets 54060, bytes 5087410, dropped 2, multicast packets 26763
    input errors 0, length 0, overrun 0, CRC 0, frame 0, fifo 0, missed 0
    output packets 13694, bytes 1282604, dropped 0
    output errors 0, aborted 0, carrier 0, fifo 0, heartbeat 0, window 0
    collisions 0

```

In this example I want to see what happening on vNic\_0. The following command will display all the captured packets to the screen. I would advise (and officially so does VMware) against running this command in a production environment with a lot of traffic as it will spew out a lot of data to the screen and can potentially cause performance issues on the ESG.

```
debug packet display interface vNic_0
```

Instead of displaying the output to the screen, you can save the output in a capture file using the following command.

```
vShield-edge-3-0> debug packet capture interface vNic_0
/blue_lane/bin/run_tcpdump: line 24: kill: (25763) - No such process
tcpdump: listening on vNic_0, link-type EN10MB (Ethernet), capture size 65535 bytes
```

You can even see that under the covers its running tcpdump, which means you can also write cool expressions or filters.

The following command excludes SSH connections to/from my IP address (10.29.16.70) from appearing in the capture. You must use an underscore between words in the expression.

```
vShield-edge-3-0> debug packet display interface vNic_0 not_port_22_and_not_host_10.29.17.60                  
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on vNic_0, link-type EN10MB (Ethernet), capture size 65535 bytes
05:49:51.551162 IP 10.29.64.240 > 10.29.16.70: ICMP echo reply, id 26207, seq 19278, length 64
05:49:51.980973 IP 10.29.2.254 > 10.29.16.70: ICMP echo reply, id 36695, seq 19879, length 64
```

After performing the capture to a file, you can list all the capture files using the following command

```
vShield-edge-3-0> debug show files
total 1.0K
-rw------- 1 708 Mar 16 05:45 tcpdump_vNic_0.0
```

That's all good, but doing a directory list doesn't really help me read the file, so to copy it off you need to use one of the following commands based on the type of transfer protocol you want to use. The choices are SCP or FTP. The following is an example of how to use SCP to copy the capture file off the ESG.

```
vShield-edge-3-0> debug copy scp sneaku@10.29.4.1:/Users/sneaku/tcpdump_vNic_0.0 tcpdump_vNic_0.0
The authenticity of host '10.29.4.1 (10.29.4.1)' can't be established.
RSA key fingerprint is c3:63:18:0b:a8:c0:f0:ed:5b:44:db:ae:61:db:9b:b6.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.29.4.1' (RSA) to the list of known hosts.
Password:
tcpdump_vNic_0.0                             100%  864     0.8KB/s   00:00
```

If you prefer, or need to use FTP, just replace the protocol choice SCP in the command with FTP.

Once you have the file off the ESG and in a location you can access, you can open the capture file with Wireshark.

Happy packet capturing!
