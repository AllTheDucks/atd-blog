+++
date = "2016-01-05T17:39:00+10:00"
draft = false
title = "How to automate the changes to the Blackboard Vagrant VM (October 2014 release)"
description = "A provisioning script to make it easier to re-provision your Blackboard vagrant VM."
keywords = ["blackboard", "vagrant", "provisioning", "October 2014", "201410"]
author = "wiley"
+++

*By Wiley Fuller.*

By now you've probably read, and used, Shane's blog posts on how to configure the Blackboard Vagrant VM so that it works properly.  However, if you've ever blown away your VM and started again from scratch, you've probably discovered that you have to go and re-apply all those changes.  This isn't such a big deal, but if you have to do it more than once or twice it gets to be a pain, and it's a serious disincentive to rolling back to a clean VM.  So, what to do. 

## Vagrant provisioners to the rescue.

To avoid all the manual work in applying the changes, we can write a shell script which will be run by Vagrant during the provisioning phase. 

The magic statement to add to your Vagrantfile, is `config.vm.provision "shell", inline: $script`.   In context, it will look like. 

````
Vagrant.configure("2") do |config|

  ...

  config.vm.provision "shell", inline: $script

end
````
I've removed a few lines for clarity. 
In this case, `$script` is a ruby variable which holds the contents of our provisioning script.   You can specify an external file if that's what floats your boat, but in order to keep everything in one place, and the number of steps to a minimum, I've just dumped it all into the `$script` variable.


## The complete Vagrantfile, what you came for in the first place.

Here's the full contents of the Vagrantfile.  It's configured to use 4096MB of RAM, so if you don't have much memore, you might want to change that.


````
# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT

echo "####################################################"
echo "Adding some useful Aliases... "
echo "####################################################"
echo " "

echo "alias tailstdout='tail -f /usr/local/blackboard/logs/tomcat/\\\`ls -1 --sort=time /usr/local/blackboard/logs/tomcat | grep stdout-stderr | head -n1\\\`'" >> /home/vagrant/.bashrc

echo "alias restartbb='sudo /usr/local/blackboard/tools/admin/ServiceController.sh services.restart'" >> /home/vagrant/.bashrc
echo "alias stopbb='sudo /usr/local/blackboard/tools/admin/ServiceController.sh services.stop'" >> /home/vagrant/.bashrc
echo "alias startbb='sudo /usr/local/blackbaord/tools/admin/ServiceController.sh services.start'" >> /home/vagrant/.bashrc
echo "alias pushbbconfig='sudo /usr/local/blackboard/tools/admin/PushConfigUpdates.sh'" >> /home/vagrant/.bashrc

echo "####################################################"
echo "Fixing Wrapper timeout and postgress connection... "
echo "####################################################"
echo " "

sudo sed -i -e 's/wrapper\.ping\.timeout=60/wrapper\.ping\.timeout=0/g' /usr/local/blackboard/config/tomcat/conf/wrapper.conf
sudo sed -i -e 's/wrapper\.ping\.timeout=60/wrapper\.ping\.timeout=0/g' /usr/local/blackboard/config/tomcat/conf/wrapper.conf.bb
sudo printf "\n\nhost    all             all             10.0.0.0/8              password\n" >>  /var/lib/pgsql/9.3/data/pg_hba.conf

sudo /etc/init.d/postgresql-9.3 reload

echo "####################################################"
echo "Waiting for Blackboard to start..."
echo "Please be patient, this could take 10 minutes or so."
echo "####################################################"
echo " "
sleep 60
curl --connect-timeout 1200 localhost:8080 &>/dev/null

echo "####################################################"
echo "Installing starting block..."
echo "####################################################"
echo " "

sudo /usr/local/blackboard/tools/admin/B2Manager.sh -i /usr/local/blackboard/system/autoinstall/internal.developer/allavailable/starting-block.war
sudo /usr/local/blackboard/tools/admin/B2Manager.sh -s AVAILABLE bb-starting-block

echo "####################################################"
echo "Restarting Blackboard..."
echo "####################################################"
echo " "


sudo /usr/local/blackboard/tools/admin/ServiceController.sh services.restart



sudo date > /etc/vagrant_provisioned_at
SCRIPT


Vagrant.configure("2") do |config|

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "4096"]
  end

  config.vm.box = 'bb-learn-9.1.201410.160373'
  config.vm.box_url = './bb-learn-9.1.201410.160373.box'

  config.vm.network :forwarded_port, guest: 8080, host: 9876
  config.vm.network :forwarded_port, guest: 8443, host: 9877
  config.vm.network :forwarded_port, guest: 2222, host: 9878
  config.vm.network :forwarded_port, guest: 5432, host: 9879

  config.vm.provision "shell", inline: $script

end
````