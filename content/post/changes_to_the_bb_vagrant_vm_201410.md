+++
date = "2015-12-09T17:39:00+10:00"
draft = false
title = "The first things you should do after downloading the Blackboard Vagrant VM (October 2014 release)"
description = "Blackboard provides a Vagrant VM for developer, but some settings should be changed to make it more useful. Updated for the October 2014 release."
keywords = ["blackboard", "vagrant", "October 2014", "201410"]
author = "shane"
+++

*By Shane Argo.*

*A while back I wrote about [the changes we make to the Blackboard Developer VM]({{< ref "post/changes_to_the_bb_vagrant_vm.md" >}}). That was written within the context of the April 2014 release. While it is still mostly relevant in the October 2014 release, there has been some changes. This is a copy of the original article with some updates. I'm hoping the 2015 Quarter 4 developer VM will have some or all of these changes already applied.*

Since Blackboard v9.1 Service Pack 14, the company has provided for download an [Oracle VirtualBox VM](https://www.virtualbox.org/) wrapped up in a nice [Vagrant VM](https://www.vagrantup.com/). This is extremely convenient, but there are a number of changes that I like to make to it immediately after downloading which makes it even more useful. These changes are: 

* Disable the wrapper timeout: When Java debugging, it's a good idea to disable the wrapper timeout. Blackboard includes a "wrapper" which, among other things, monitors the status of the application. If the application stops responding for a specified period of time the wrapper restarts it. Useful in production, very inconvenient when debugging because the application stops responding when it hits a breakpoint in your code, thus triggering the wrapper to restart it. 
* Allow external database connections: Quite often, it is useful to connect to Blackboard's database management system.
* Increase the memory: The development VM comes with 1024mb of memory allocated by default. I find this isn't quite enough, and prefer increase it slightly for better performance.
* Install the Starting Block.


# Vagrantfile #

The first changes we'll make are to the Vagrantfile itself. This file defines details about the virtual machine and it's import into Oracle VirtualBox. We'll use it to increase the memory allocation. Edit the Vagrantfile you downloaded from Blackboard's portal. 

Under the port forwarding lines we'll add the following:

````
config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "1536"]
end
````

This instructs Oracle VirtualBox to assign 1.5gb of memory to the machine, instead of the default 1gb.


# Blackboard configuration files #

We'll now modify some of the configuration files in Blackboard. Start up the VM and connect to it with SSH (this is out of scope for this post).


## wrapper.conf.properties ##

The wrapper configuration is in the file /usr/local/blackboard/config/tomcat/conf/wrapper.conf.bb. We'll now disable the timeout by changing the following setting (a zero timeout means disabled):

````
wrapper.ping.timeout=0
````

We'll now need to run PushConfigUpdate.sh script to apply this change:
````
sudo /usr/local/blackboard/tools/admin/PushConfigUpdates.sh.
````

## pg_hba.conf ##

PostgreSQL's default configuration prevents connections from externally. We'll need to change this so we can connect using a client. The configuration file we need to modify is /var/lib/pgsql/9.3/data/pg_hba.conf.

Add this line to the file:

````
host    all             all             10.0.0.0/8              password
````

We must now signal to PostgreSQL to reload the configuration file we just changed:

````
sudo /etc/init.d/postgresql-9.3 reload
````

# Install the Starting Block #

The developer VM is meant to have the *Starting Block* building block installed. For some reason the October 2015 release Developer VM missed out. Thankfully the building block package is inside the VM so we can easily install it using the B2Manager.sh script:

````
sudo /usr/local/blackboard/tool/admin/B2Manager.sh -i /usr/local/blackboard/system/autoinstall/internal.developer/allavailable/starting-block.war
sudo /usr/local/blackboard/tool/admin/B2Manager.sh -s AVAILABLE bb-starting-block
````


# Summary #

Thats it! We've increased the memory allocation. We've disabled the wrapper timeout so that we can debug in peace and we've opened up the database for access using something like [pgAdmin](http://www.pgadmin.org/). We also installed the missing starting block.
