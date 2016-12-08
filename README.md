ansible-windows-springboot
======================================================================================

# Example project showing how to provision, deploy and run a Spring Boot app as a Windows Service using Ansible, chocolatey &amp; nssm

There are times, when you are forced to use a Windows box instead of a smoothly running Linux for your Spring Boot app to run on - e.g., if you have to call native libraries like C/C++ programs packaged as exe or dll (you can use [JNA](https://github.com/java-native-access/jna) and [JNAerator](https://github.com/nativelibs4java/JNAerator) for the integration stuff). 

You know Windows from ancient times, maybe also from you´re big home gaming machine. But maybe you´re just like me: bringing together Windows and modern software development doesn´t feel as there´s a overlap, right?! Well - after beeing in the situation mentioned above - that doesn´t mean, it´s not possible. It´s the other way around! Because we can use our well known CI Server (like [Jenkins](https://jenkins.io/)) to really manage Windows boxes with the help of our beloved [Ansible](https://www.ansible.com/)! Do you like this idea? Let´s go ahead and try that out!

# Setup