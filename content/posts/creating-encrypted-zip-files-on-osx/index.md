---
title: "Creating encrypted zip files on OSX"
date: 2015-05-24
coverImage: "finder_logo.png"
---

Its become quite a common occurance that I need to create encrytped zip files to send to clients, and it seems to be one of those things which I always have to Google to figure out how to do it.

So to save me looking up Google all the time, I am posting it up here so I always know where to look for it.

This command will create an encrypted zip file of all files in a folder, exlcuding those pesky hidden OSX ones that begin with a period (.)

```
zip -e encryptedZipFile.zip folderToCompressAndEncrypt/* -x"^.*"

```

When you run it, it should look something similar to this.

```
zip -e encryptedZipFile.zip folderToCompressAndEncrypt/* -x"^.*"
Enter password: 
Verify password: 
  adding: folderToCompressAndEncrypt/file1.txt (deflated 76%)
  adding: folderToCompressAndEncrypt/file2.txt (deflated 87%)
```

And to unzip, you can just open the file in Finder, which will prompt you for the password, or use the following command to unzip the file in Terminal:

```
unzip encryptedZipFile.zip 

```

Which should prompt you for the password and then unzip the files:

```
unzip encryptedZipFile.zip 
Archive:  encryptedZipFile.zip
[encryptedZipFile.zip] encryptedZipFile/file1.txt password: 
  inflating: encryptedZipFile/file1.txt  
  inflating: encryptedZipFile/file2.txt
```

To recursively zip everything in a folder, excluding the annoying .DS\_Store files, and specific sub-directories, its possible to use the following command:

```
zip -r Archive.zip * --exclude=*testing* --exclude=*dev* --exclude=*.DS_Store* --exclude=*.zip
```
