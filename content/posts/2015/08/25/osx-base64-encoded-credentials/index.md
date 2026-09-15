---
title: "OSX Base64 Encoded Credentials"
date: 2015-08-25
categories: 
  - "nsx"
  - "vmware"

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
cover:
  image: "images/Yosemite-OS-X-Mac_276x1000.jpg"
  relative: true
  hiddenInList: false
  hiddenInSingle: true
---

For those who have used the NSX vSphere REST API, you will know that with every API call you are required to send the following in the headers:

```
Content-Type      application/xml
Basic             <base64 encoded credentials>
```

So if you have an OSX machine, you can leverage openssl (which is installed by default) to generate the base64 encoded credentials. This means you do not have to find a website and paste credentials into the site to generate the encoded credentials.

The command is as follows

```
echo -n '<username:password>' | openssl base64
```

So to generate the credentials for a default username of admin and a password of 'default', you would perform the following:

```
echo -n 'admin:default' | openssl base64
```

Which will spit out the credentials

```
SneakU$ echo -n 'admin:default' | openssl base64
YWRtaW46ZGVmYXVsdA==
SneakU$
```
