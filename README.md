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
get-executionpolicy
set-executionpolicy remotesigned
```

If that´s not enough, try the harder one:
```
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
```

#### Upgrade to Powershell 3.0

If the following command shows something < 3.0:
```
get-host
```

, you should upgrade with https://github.com/cchurch/ansible/blob/devel/examples/scripts/upgrade_to_ps3.ps1 (this will [reboot your Windows box!](http://serverfault.com/questions/539229/possible-to-upgrade-powershell-2-0-to-3-0-without-a-reboot) )

#### Configure remote access for ansible

Run https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1


#### Testdrive Ansible connectivity
```
ansible yourPlayBookNameWithOutDotYml -i hostsfile -m win_ping
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