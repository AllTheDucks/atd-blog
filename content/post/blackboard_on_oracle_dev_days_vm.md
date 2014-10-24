+++
date = "2013-10-31T11:49:53+11:00"
draft = false
title = "Blackboard on Oracle Dev Days VM"
author = "wiley"
keywords = ["blackboard", "oracle", "virtualbox", "vm"]
+++

I hate installing Blackboard, but I hate installing and configuring Oracle even more.  As a result, I find the easiest way to get a Blackboard Development VM up and running, is to install Bb into the [Oracle Dev Days VM](http://www.oracle.com/technetwork/database/enterprise-edition/databaseappdev-vm-161299.html) which is freely downloadable from the Oracle site.   Here’s the steps I follow:

1. Install VirtualBox if you don’t have it.
2. Download and install Oracle DevDays VM with all the oracle stuff.
  1. [http://www.oracle.com/technetwork/database/enterprise-edition/databaseappdev-vm-161299.html](http://www.oracle.com/technetwork/database/enterprise-edition/databaseappdev-vm-161299.html)
  2. you’ll need an oracle account, which is free to set up.
  3. All the usernames and passwords for the VM are “oracle” and
3. Disable the oracle HTTP service running on port 80, and restart oracle.
  1. Edit /home/oracle/app/oracle/product/11.2.0/dbhome_2/network/admin/listener.ora and remove the entry for the listener on port 80.
  2. su -
  3. /etc/init.d/oracle restart
4. Install Java from Oracle JDK RPM, as root.
  1. rpm -Uvh jdk-7u25-linux-i586.rpm
5. Download Blackboard from Behind the Blackboard in the VM.
6. Create a bbuser account using the users and groups tool, and make sure you create a private group for that user.
7. Create the directories for blackboard
  1. home directory ````/usr/local/blackboard````
    1. ````mkdir /usr/local/blackboard````
	2. ````chown bbuser:bbuser /usr/local/blackboard````
  2. bb content directory /usr/local/blackboard/content
    1. ````mkdir /usr/local/blackboard/content````
    2. ````chown bbuser:bbuser /usr/local/blackboard/content````
  3. oracle data directory /usr/local/bboracle (user:oracle group:oracle)
    1. mkdir /usr/local/bboracle
    2. chown oracle:oracle /usr/local/bboracle
8. run the blackboard installer *AS ROOT*.
  1. All the database usernames and passwords should be “*oracle*” and “*oracle*”
  2. use the paths above when prompted.
If you have trouble getting blackboard to listen on port 80, you can either change it to port 8080 by setting the following property in ````/usr/local/blackboard/config/bb-config.properties````
````bbconfig.unix.httpd.portnumber=8080````
, and then running ````usr/local/blackboard/tools/admin/PushConfigUpdates.sh```` 


