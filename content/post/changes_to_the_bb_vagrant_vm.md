+++
date = "2014-10-27T17:51:00+10:00"
draft = false
title = "The first things you should do after downloading the Blackboard Vagrant VM"
description = "Blackboard provides a Vagrant VM for developer, but some settings should be changed to make it more useful."
keywords = ["blackboard", "vagrant"]
+++

Since Blackboard v9.1 Service Pack 14, the company has provided for download an [Oracle VirtualBox VM](https://www.virtualbox.org/) wrapped up in a nice [Vagrant VM](https://www.vagrantup.com/). This is extremely convenient, but there are a number of changes that I like to make to it immediately after downloading which makes it even more useful. These changes are: 

* Disable SSL: SSL ([technically TLS](https://luxsci.com/blog/ssl-versus-tls-whats-the-difference.html)) is an extremely important technology for a production system, but is hardly necessary while developing a building block. The main reason to disable it is because the "Starting Block" and "DeployB2" Apache Ant task do not work while it's enabled.
* Disable the wrapper timeout: When Java debugging, it's a good idea to disable the wrapper timeout. Blackboard includes a "wrapper" which, among other things, monitors the status of the application. If the application stops responding for a specified period of time the wrapper restarts it. Useful in production, very inconvenient when debugging because the application stops responding when it hits a breakpoint in your code, thus triggering the wrapper to restart it. 
* Allow external database connections: Quite often, it is useful to connect to Blackboard's database management system.
* Increase the memory: The development VM comes with 1024mb of memory allocated by default. I find this isn't quite enough, and prefer increase it slightly for better performance.


# Vagrantfile #

The first changes we'll make are to the Vagrantfile itself. This file defines details about the virtual machine and it's import into Oracle VirtualBox. We'll use it to forward additional ports and increase the memory allocation. Edit the Vagrantfile you downloaded from Blackboard's portal. There isn't much in there, but you'll notice a number of lines like this:

````
config.vm.network :forwarded_port, guest: 2222, host: 9878
````

This is defining a port forwarding, from 2222 to 9878. 2222 is the Java debugging port, and it's being forwarded to 9878. This means you can attach th Java debugger to the address localhost:9878 to debug you building blocks.

 Let's define a few more port forwardings:

````
config.vm.network :forwarded_port, guest: 8080, host: 9876
config.vm.network :forwarded_port, guest: 5432, host: 5432
````

Port 8080 is the HTTP (non-SSL) port and thus we'll connect to Blackboard on port 9876 once we've disabled SSL. Port 5432 is the PostgreSQL database port. Once we've enabled external connections to the database, this will be the port we'll connect on.

Under the port forwarding lines we'll also add the following:

````
config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "1536"]
end
````

This instructs Oracle VirtualBox to assign 1.5gb of memory to the machine, instead of the default 1gb.


# Blackboard configuration files #

We'll now modify some of the configuration files in Blackboard. Start up the VM and connect to it with SSH (this is out of scope for this post).


## bb-config.properties ##

Change the following keys within the /usr/local/blackboard/config/bb-config.properties file:

````
bbconfig.frontend.portnumber=9876
...
bbconfig.frontend.protocol=http
````

The first setting tells the application which port we'll be using to access Blackboard (the port we configured for HTTP). The second tells it to use HTTP instead of HTTPS.

## wrapper.conf.properties ##

The wrapper configuration is in the file /usr/local/blackboard/config/tomcat/conf/wrapper.conf.bb. We'll now disable the timeout by changing the following setting (a zero timeout means disabled):

````
wrapper.ping.timeout=0
````

## Apply the configuration changes ##

We'll now need to run PushConfigUpdate.sh script:
````
sudo /usr/local/blackboard/tools/admin/PushConfigUpdates.sh.
````

## pg_hba.conf ##

PostgreSQL's default configuration prevents connections from externally. We'll need to change this so we can connect using a client. The configuration file we need to modify is /var/lib/pgsql/9.2/data/pg_hba.conf.

Add this line to the file:

````
host    all             all             10.0.0.0/8              password
````

We must now signal to PostgreSQL to reload the configuration file we just changed:

````
sudo /etc/init.d/postgresql-9.2 reload
````

# Summary #

Thats it! We've disabled SSL and the wrapper timeout, opened up the database for access using something like [pgAdmin](http://www.pgadmin.org/) and increased the memory allocation. 