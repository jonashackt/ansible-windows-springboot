ansible-windows-springboot
======================================================================================

## Example project showing how to provision, deploy and run a Spring Boot app as a Windows Service using Ansible, chocolatey &amp; nssm

There are times, when you are forced to use a Windows box instead of a smoothly running Linux for your Spring Boot app to run on - e.g., if you have to call native libraries like C/C++ programs packaged as exe or dll (you can use [JNA](https://github.com/java-native-access/jna) and [JNAerator](https://github.com/nativelibs4java/JNAerator) for the integration stuff). 

You know Windows from ancient times, maybe also from you´re big home gaming machine. But maybe you´re just like me: bringing together Windows and modern software development doesn´t feel as there´s a overlap, right?! Well - after beeing in the situation mentioned above - that doesn´t mean, it´s not possible. It´s the other way around! Because we can use our well known CI Server (like [Jenkins](https://jenkins.io/)) to really manage Windows boxes with the help of our beloved [Ansible](https://www.ansible.com/)! Do you like this idea? Let´s go ahead and try that out!

Ansible nutzt für die Connectivity mit Windows kein ssh, da dies auf Windows nicht verfügbar ist, sondern WinRM (Windows Remote
Management) - siehe http://www.it-visions.de/glossar/alle/4418/Windows_Remote_Management_WinRM.aspx


## Prerequisites

We need a Windows box to do show everything. So if you don´t have one spare, go to https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/#downloads and download an Vagrant image with Windows 10 (e.g. for VirtualBox - be sure to have the VM-Provider installed). You should get something like a MSEdge.Win10_RS1.Vagrant.zip - extract it (Mac: with the [Unarchiver](http://wakaba.c3.cx/s/apps/unarchiver.html)) and there you are: The Windows Vagrant box __dev-msedge.box__ is ready :)

Because Microsoft doesn´t seem to ship metadata for the box, add it to Vagrant via:

```
vagrant box add dev-msedge.box --name "windows10"
```

I added a Vagrantfile to this repository, so you can start right away by (if you have Vagrant installed ;) )
```
vagrant up
```

Because we use Windows, SSH for example will not work and we need to tweak some Vagrant Configuration Options described here: https://www.vagrantup.com/docs/vagrantfile/winrm_settings.html


## Prepare the Windows Box for Ansible Communication

Doku: http://docs.ansible.com/ansible/intro_windows.html#windows-system-prep

### prepare the Windows box

#### Allow execution of scripts on Powershell:
```
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
```

> If you have Windows >= v10, just skip the follwing steps & proceed with Configure remote access for ansible

#### Upgrade to Powershell 3.0

If __get-host__ shows something < 3.0, you should upgrade with https://github.com/cchurch/ansible/blob/devel/examples/scripts/upgrade_to_ps3.ps1 (this will [reboot your Windows box!](http://serverfault.com/questions/539229/possible-to-upgrade-powershell-2-0-to-3-0-without-a-reboot) )

#### Configure remote access for ansible

Run
Download the Script https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1 and run it from Powershell (with Admin rights).


#### Testdrive Ansible connectivity
```
cd spring-boot-playbook
ansible restexample-windows-dev -i hostsfile -m win_ping
```


## Choose an Spring Boot app to deploy

This is the easy task - we want to deploy a Spring Boot app. So just fire up one in Minutes e.g. with [Spring Initializr](http://start.spring.io/), choose an existing one you have ready to build or just take the simple project here: https://github.com/jonashackt/restexamples

Either way you choose: Be sure to have a working Build in Place, so that you have a runnable Spring Boot jar-File in place (e.g. restexamples-0.0.1-SNAPSHOT.jar). For the example project [restexamples](https://github.com/jonashackt/restexamples) you get this by running:
```
mvn clean package
```


## Craft a Windows-ready ansible playbook

Let´s run our the playbook restexample-windows.yml:

```
ansible-playbook -i hostsfile restexample-windows.yml --extra-vars "service_jar=../../restexamples/target/restexamples-0.0.1-SNAPSHOT.jar service_name=restexample-springboot host=restexample-windows-dev" --check
```




If this brings you something like the following output, your Windows Box is ready for Ansible:
```
user@macbook:/yourPathHere/ansibleproject$ ansible yourPlayBookNameWithOutDotYml -i hostsfile -m win_ping
192.168.10.99 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```



## Best practices

* __ALWAYS__: escape \ in paths with "\" --> e.g. C:\\\temp
* don´t assume that a path with C:\ProgamFiles (x86)\XYZ will work (e.g. for Java better use "C:\\\ProgramData\\\Oracle\\\Java\\\javapath\\\java.exe")
* if chocolatey doesn´t want to work, you have to install it once manually on your Windows box
```
iwr https://chocolatey.org/install.ps1 -UseBasicParsing | iex
```