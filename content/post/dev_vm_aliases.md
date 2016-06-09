+++
date = "2016-06-08T16:35:00+10:00"
draft = false
title = "Useful aliases in the Blackboard Dev VM"
description = "Create Bash aliases in the Blackboard Dev VM to use as shortcuts to common commands."
keywords = ["blackboard", "dev vm", "bash", "aliases"]
+++

*By Shane Argo.*

When developing extensions to Blackboard using the Blackboard Developer VM, there are a number of shell commands that you'll find yourself using frequently. Perhaps the most common is tailing the latest stdout-stderr log, but you'll also want to start, stop and restart Blackboard, and push configuration changes quite often.

As I found myself constantly doing this, I've created a number of Bash aliases so I can run these commands with a single command without needing to change directory or specify full paths.

To use these aliases, you'll need to add them to your .bashrc file. This is out of scope for this post, but there are lots of tutorials online.

## Tail stdout-stderr ##
````bash
alias tailstdout='tail -f /usr/local/blackboard/logs/tomcat/\\\`ls -1 --sort=time /usr/local/blackboard/logs/tomcat | grep stdout-stderr | head -n1\\\`'
````

This is my favourite. When I execute `tailstdout` from any directory, it finds the most recently modified stdout-stderr log file in Blackboard's logs/tomcat directory and begins tailing it.

*Bonus*: Bash aliases are simple text substitutions. That is to say, the word `tailstdout` is simply replaced verbatim with the text defined by the alias before executing the command. So you can actually pass additional flags to tail if you wish. Want to see the last one thousand lines of output in the log simply run `tailstdout -n1000`.

## Starting, Stopping and Restarting Blackboard ##
````bash
alias restartbb='sudo /usr/local/blackboard/tools/admin/ServiceController.sh services.restart'
alias stopbb='sudo /usr/local/blackboard/tools/admin/ServiceController.sh services.stop'
alias startbb='sudo /usr/local/blackboard/tools/admin/ServiceController.sh services.start'
````

These aliases are a lot simpler. They simply execute the commands to start, stop and restart Blackboard, but using the absolute path to the scripts allowing them to be run from anywhere.

## Pushing Blackboard Configuration ##
````bash
alias pushbbconfig='sudo /usr/local/blackboard/tools/admin/PushConfigUpdates.sh'
````

Similarly to the start, stop and restart aliases, I find it useful to be able to push configuration changes from anywhere with a simple command.

## Automatically adding the aliases to the Dev VM ##
I try not to make too many changes to the Dev VM from the way it's delivered by Blackboard. This is so that I can simply destroy and recreate the VM without creating too much work. When I do add things, I like to automate it if possible. 

Wiley blogged a little while back about [modifying the Vagrantfile delivered by Blackboard to automatically make a number of very useful changes upon provisioning the VM]({{< ref "post/vagrant_vm_provisioner_script_for_201410.md" >}}). Thankfully, Blackboard began making these changes to the VM prior to distribution. That is, with the exception of these aliases. So borrowing Wiley's technique, we can automatically create these aliases upon provisioning the VM by modifying the Vagrant file:

````ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT

echo "alias tailstdout='tail -f /usr/local/blackboard/logs/tomcat/\\\`ls -1 --sort=time /usr/local/blackboard/logs/tomcat | grep stdout-stderr | head -n1\\\`'" >> /home/vagrant/.bashrc

echo "alias restartbb='sudo /usr/local/blackboard/tools/admin/ServiceController.sh services.restart'" >> /home/vagrant/.bashrc
echo "alias stopbb='sudo /usr/local/blackboard/tools/admin/ServiceController.sh services.stop'" >> /home/vagrant/.bashrc
echo "alias startbb='sudo /usr/local/blackboard/tools/admin/ServiceController.sh services.start'" >> /home/vagrant/.bashrc
echo "alias pushbbconfig='sudo /usr/local/blackboard/tools/admin/PushConfigUpdates.sh'" >> /home/vagrant/.bashrc

SCRIPT

Vagrant.configure("2") do |config|
  ...
  config.vm.provision "shell", inline: $script
  ...
end
````

