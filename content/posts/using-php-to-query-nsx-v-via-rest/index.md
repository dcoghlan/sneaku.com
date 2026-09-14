---
title: "Using PHP to query NSX-v via REST"
date: 2015-11-09
categories: 
  - "nsx"
  - "vmware"
tags: 
  - "nsx"
  - "vmware"
coverImage: "php_square_512x512.png"
---

Whilst working on a little side project of mine, I wanted to be able to submit a REST API call against a NSX Manager. At first I was just going to use shell\_exec to execute one of my python scripts, but after a bit of investigation, I found that I could use the PHP Client URL Library ([cURL](http://php.net/manual/en/book.curl.php)) .

It took me a little bit of time to get it working just the way I wanted, but now that I have the basics, it should be simple to be able to do most things through a webpage.

Here is a sample webpage which adds syslog information to a NSX Controller.

Everything is pretty much hard coded, but being php you could do almost anything you want.... like submit a form with the details.

Now I have figured this one out, I will be working out how to do it with AJAX/jQuery and will post up a sample working config once I figure it out.

Happy Coding!

```
<?php

// NSX Manager IP address or Fully Qualified Domain Name
$nsxmgr = "10.10.128.123";

// Credentials required to access NSX Manager API
$username = "admin";
$password = "VMware1!";

// NSX Controller ID
$controllerId = 'controller-1';

// Syslog server details
$syslogServer = '10.10.10.10';
$port = '514';
$protocol = 'UDP';
$level = 'INFO';

// Set debug mode (True or False)
$debugMode = False;

// *****************************************************************************
// No need to change anything below this line.
// *****************************************************************************

// Create the credentials in the required layout
$credentials = $username . ":" . $password;

// Set NSX Headers that are required
$headers = array(
    'Content-Type: application/xml',
    'Authorization: Basic ' . base64_encode($credentials)
);

// Generate the XML to submit via the NSX Manager API
$input_xml = <<<XML
<controllerSyslogServer>
    <syslogServer>$syslogServer</syslogServer>
    <port>$port</port>
    <protocol>$protocol</protocol>
    <level>$level</level>
</controllerSyslogServer>
XML;

// DEBUGGING: Print the generated XML
if ($debugMode == True)
{
    print_r("<br><pre>" . htmlentities($input_xml) . "</pre>");
}

// Set the API URL
$api_url = 'https://' . $nsxmgr . '/api/2.0/vdn/controller/' . $controllerId . '/syslog';

// Initialise a curl connection
$curl = curl_init();

// Set some options for how curl needs to behave
curl_setopt($curl, CURLOPT_RETURNTRANSFER, false);
curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, false);
curl_setopt($curl, CURLOPT_FAILONERROR, false);
curl_setopt($curl, CURLOPT_URL, $api_url);
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, $input_xml);
curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
curl_setopt($curl, CURLOPT_HEADER, false);
curl_setopt($curl, CURLOPT_TIMEOUT, 10);

// Execute the curl request
$response = curl_exec($curl);

// Get the infomation regarding the request
$info = curl_getinfo($curl);

// Check to see if there is a error, and if so, display the error.
if(curl_errno($curl))
{
    print_r('<br>');
    print_r(str_repeat('*',79));
    print_r('<br>');
    print_r('CURL ERROR: ' . curl_error($curl) . '<br>');
    print_r('Check the details at the top of the page.<br>');
}

// Print the array for the information about the curl request
print_r('INFO:<pre>');
print_r($info);
print_r('</pre><br>');

// Close the curl connection
curl_close($curl);

?>
```
