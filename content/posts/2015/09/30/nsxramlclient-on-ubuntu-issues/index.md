---
title: "NSXRAMLCLIENT on Ubuntu Issues"
date: 2015-09-30
categories: 
  - "nsx"
  - "scripts"
  - "vmware"

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true
cover:
---

Following on from my last post about the [NSXRAMLCLIENT](http://www.sneaku.com/2015/09/18/nsx-vsphere-raml-client/), I decided to setup an Ubuntu machine dedicated to being able to run the NSXRAMLCLIENT in my home lab.

Using my own instructions that I had written previously to get it up and running on my OSX 10.10.5 machine, I came across a peculiar issue when trying to initiate a connection to the nsx manager.

Here is the code I was running which works on my OSX machine without issues:

```
from nsxramlclient.client import NsxClient

nsxraml_file = '/home/user/nsxraml/nsxraml-master/nsxvapiv614.raml'
nsxmanager = '10.10.128.123'
nsx_username = 'admin'
nsx_password = 'default'

client_session = NsxClient(nsxraml_file, nsxmanager, nsx_username, nsx_password, verify=False)

ssoConfig = client_session.read('ssoConfig')
print ssoConfig
```

But when running on my Ubuntu 14.04.3 LTS box, this was the result:

```
user@nsxraml:~/nsxraml$ python test.py 
Traceback (most recent call last):
  File "/home/user/nsxraml/test.py", line 10, in <module>
    client_session = NsxClient(nsxraml_file, nsxmanager, nsx_username, nsx_password, verify=False)
  File "/usr/local/lib/python2.7/dist-packages/nsxramlclient/client.py", line 57, in __init__
    self._suppress_warnings)
  File "/usr/local/lib/python2.7/dist-packages/nsxramlclient/http_session.py", line 69, in __init__
    requests.packages.urllib3.disable_warnings()
AttributeError: 'module' object has no attribute 'packages'
```

As it turns out, the problem is that somewhere along the line, APT has been used to install the python-requests module. Even though PIP was used later to install the NSXRAMLCLIENT module, which downloads and installs the latest version of the python requests module as a Python distribution module, the version installed by APT will take precedence over the version installed by PIP.

To verify that APT has installed the python-requests module, issue the following command. If it comes back with output similar to below, you know APT has installed the module:

```
dpkg -s python-requests
```

```
user@nsxraml:~/nsxraml$ dpkg -s python-requests
Package: python-requests
Status: install ok installed
Priority: optional
Section: python
Installed-Size: 210
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Architecture: all
Source: requests
Version: 2.2.1-1ubuntu0.3
Depends: python:any (<< 2.8), python:any (>= 2.7.5-5~), ca-certificates, python-chardet, python-urllib3 (>= 1.7.1)
Description: elegant and simple HTTP library for Python, built for human beings
 Requests allow you to send HTTP/1.1 requests. You can add headers, form data,
 multipart files, and parameters with simple Python dictionaries, and access the
 response data in the same way. It's powered by httplib and urllib3, but it does
 all the hard work and crazy hacks for you.
 .
 Features
 .
   - International Domains and URLs
   - Keep-Alive & Connection Pooling
   - Sessions with Cookie Persistence
   - Browser-style SSL Verification
   - Basic/Digest Authentication
   - Elegant Key/Value Cookies
   - Automatic Decompression
   - Unicode Response Bodies
   - Multipart File Uploads
   - Connection Timeouts
Homepage: http://python-requests.org
Original-Maintainer: Debian Python Modules Team <python-modules-team@lists.alioth.debian.org>
```

You can see above that APT has installed version 2.2.1 of the Python Requests module.

So we can simply remove the module:

```
sudo apt-get remove python-requests
```

```
user@nsxraml:~/nsxraml$ sudo apt-get remove python-requests
sudo: unable to resolve host nsxraml
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  python-chardet-whl python-colorama python-colorama-whl python-distlib
  python-distlib-whl python-html5lib python-html5lib-whl python-pip-whl
  python-requests-whl python-setuptools-whl python-six-whl python-urllib3-whl
  python-wheel python3-pkg-resources
Use 'apt-get autoremove' to remove them.
The following packages will be REMOVED:
  python-pip python-requests
0 to upgrade, 0 to newly install, 2 to remove and 3 not to upgrade.
After this operation, 692 kB disk space will be freed.
Do you want to continue? [Y/n] Y
(Reading database ... 68806 files and directories currently installed.)
Removing python-pip (1.5.4-1ubuntu3) ...
Removing python-requests (2.2.1-1ubuntu0.3) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
```

If you take a look in /usr/local/lib/python2.7/dist-packages/ you can see that requests 2.7.0 has been previously downloaded by PIP when the NSXRAMLCLIENT was installed.

```
user@nsxraml:~/nsxraml$ ls -la /usr/local/lib/python2.7/dist-packages/
total 172
drwxrwsr-x 13 root staff  4096 Sep 29 18:27 .
drwxrwsr-x 29 root staff 20480 Sep 29 18:27 ..
-rw-r--r-- 1 root staff   218 Sep 29 12:31 easy-install.pth
drwxr-sr-x  5 root staff  4096 Sep 29 18:27 lxml
drwxr-sr-x  2 root staff  4096 Sep 28 15:51 lxml-3.4.4.egg-info
drwxr-sr-x  2 root staff  4096 Sep 29 18:27 nsxramlclient
drwxr-sr-x  2 root staff  4096 Sep 29 12:31 nsxramlclient-1.0.3.egg-info
drwxr-sr-x  2 root staff  4096 Sep 29 18:27 pyraml
drwxr-sr-x  2 root staff  4096 Sep 28 15:47 pyraml_parser-0.1.5.egg-info
drwxr-sr-x  2 root staff  4096 Sep 28 15:51 PyYAML-3.11.egg-info
drwxr-sr-x  3 root staff  4096 Sep 30 07:42 requests
drwxr-sr-x  2 root staff  4096 Sep 30 07:42 requests-2.7.0.dist-info
-rw-r--r-- 1 root staff    33 Sep 28 16:43 setuptools.pth
drwxr-sr-x  2 root staff  4096 Sep 29 12:31 six-1.9.0.dist-info
-rw-r--r-- 1 root root  29664 Sep 29 12:31 six.py
-rw-r--r-- 1 root root  30194 Sep 29 18:27 six.pyc
-rw-r--r-- 1 root staff 30194 Sep 29 18:27 six.pyo
drwxr-sr-x  2 root staff  4096 Sep 29 18:27 yaml
```

And just confirm that the requests module is no longer installed via APT

```
user@nsxraml:~$ dpkg -s python-requests
dpkg-query: package 'python-requests' is not installed and no information is available
Use dpkg --info (= dpkg-deb --info) to examine archive files,
and dpkg --contents (= dpkg-deb --contents) to list their contents.
```

Now when I run my test script again, it completes as intended:

```
user@nsxraml:~/nsxraml$ python test.py 
OrderedDict([('status', 200), ('body', {'ssoConfig': {'vsmSolutionName': 'VSM_SOLUTION_eb3f6754-0992-4e31-86a3-0b545c6b726d', 'ssoAdminUsername': 'administrator@vsphere.local', 'ssoLookupServiceUrl': 'https://10.10.128.122:443/lookupservice/sdk'}}), ('location', None), ('objectId', None), ('Etag', None)])
```
