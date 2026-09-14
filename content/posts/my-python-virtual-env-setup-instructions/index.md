---
title: "My Python Virtual Env Setup Instructions"
date: 2016-11-17
categories: 
  - "scripts"
---

Seeing as though I am always building different Ubuntu virtual machines to perform various python actions on (often on different customer sites), I find myself setting up the same things over and over again.

My notes for this process are normally scattered between Notepad/TextEdit and Evernote, so its about time to put them all in the one spot so I can reference them easily.

Depending on the environment I am working in, I may need to do an upgrade of the Ubuntu OS.

```
sudo do-release-upgrade -f DistUpgradeViewNonInteractive
sudo reboot
```

Once the VM has been upgraded and rebooted, lets update APT.

```
sudo apt-get update && sudo apt-get upgrade
```

Now install git & git-flow

```
sudo apt-get -y install git git-flow
```

Install PIP if it is not already installed

```
apt-get -y install python-pip
```

If PIP is already installed, maybe update ip

```
sudo pip install --upgrade pip
```

Install virtualenv via PIP

```
sudo pip install virtualenv
```

Create a directory which will store the virtual environments

```
mkdir ~/.virtualenvs
```

It is at this point that it is possible to be able to use the virtual environments now, but its a bit tedious, so we install virtualenvwrapper to make things easier.

```
sudo -H pip install virtualenvwrapper
```

Edit the ~/.bashrc file and add this to the bottom of the file

```
# The following lines are used for virtualenvwrapper

VIRTUALENVWRAPPER_PYTHON='/usr/bin/python'
export WORKON_HOME=$HOME/.virtualenvs
source /usr/local/bin/virtualenvwrapper.sh
```

Exit and re-open your shell, or reload the ~/.bashrc file with the following command and you will be ready to start using virtualenvs and virtualenvwrapper.

```
. ~/.bashrc
```

```
admin@mgt-lnxjump:~/git$ . ~/.bashrc
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/initialize
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/premkvirtualenv
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/postmkvirtualenv
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/prermvirtualenv
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/postrmvirtualenv
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/predeactivate
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/postdeactivate
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/preactivate
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/postactivate
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/get_env_details
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/premkproject
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/postmkproject

```

# Working with VirtualEnvs

## Creating a new virtualenv

```
mkvirtualenv ClientEnvironment1
```

```
admin@mgt-lnxjump:~/git$ mkvirtualenv ClientEnvironment1
New python executable in /home/admin/.virtualenvs/ClientEnvironment1/bin/python
Installing setuptools, pip, wheel...done.
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/ClientEnvironment1/bin/predeactivate
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/ClientEnvironment1/bin/postdeactivate
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/ClientEnvironment1/bin/preactivate
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/ClientEnvironment1/bin/postactivate
virtualenvwrapper.user_scripts creating /home/admin/.virtualenvs/ClientEnvironment1/bin/get_env_details

```

## Delete a virtualenv

```
rmvirtualenv ClientEnvironment1
```

```
admin@mgt-lnxjump:~/git$ rmvirtualenv ClientEnvironment1
Removing ClientEnvironment1...

```

## Show all virtualenvs configured

Use either of the following 2 commands to list all the virtualenvs configured

```
lsvirtualenv
```

```
admin@mgt-lnxjump:~/git$ lsvirtualenv
ClientEnvironment1
==================

```

```
workon
```

```
admin@mgt-lnxjump:~/git$ workon
ClientEnvironment1

```

## Use a particular virtualenv

```
workon ClientEnvironment1
```

```
admin@mgt-lnxjump:~/git$ workon ClientEnvironment1
(ClientEnvironment1) admin@mgt-lnxjump:~/git$

```

## Stop using a virtualenv

```
deactivate
```

```
(ClientEnvironment1) admin@mgt-lnxjump:~/git$ deactivate
admin@mgt-lnxjump:~/git$

```

# Using PIP

Show all python packages installed within the virtualenv

```
pip list
```

```
(ClientEnvironment1) admin@mgt-lnxjump:~/git$ pip list
DEPRECATION: The default format will switch to columns in the future. You can use --format=(legacy|columns) (or define a format=(legacy|columns) in your pip.conf under the [list] section) to disable this warning.
pip (9.0.1)
requests (2.12.1)
setuptools (28.8.0)
wheel (0.29.0)
```

```
pip list  --format=columns
```

```
(ClientEnvironment1) admin@mgt-lnxjump:~/git$ pip list  --format=columns
Package    Version
---------- -------
pip        9.0.1
requests   2.12.1
setuptools 28.8.0
wheel      0.29.0

```

To generate a list of modules that need to be installed to replicate the current environment use the following command

```
pip freeze
```

```
(ClientEnvironment1) admin@mgt-lnxjump:~/git$ pip freeze
requests==2.12.1

```

And you can also save this to a file which can then be used later to install all the modules in another environment via PIP

```
pip freeze > requirements.txt
```

```
pip install -r requirements.txt
```
