[**Linux**](../Linux.md)
# How to create a custom systemd Service in CentOS 7

In this tutorial, I am going to show you how to create services in CentOS 7. 

## To list all the Service in CentOS 7:

~~~bash
systemctl --type=service
~~~
The above command will list all the services and its state in the CentOS 7

To start, stop & check status of Service in CentOS 7:

```
systemctl start firewalld.service

systemctl stop firewalld.service

systemctl status firewalld.service
```

Create a script to run in Service:

### To create a service, then you need any Linux package or script to use at the run time of the service. So i am going to create a script that will create a file inside /tmp directory.

~~~bash
cd /
mkdir scripts
cd scripts
~~~

```
vim myservice.sh
#! /bin/bash
touch /tmp/test-service-file
```
~~~bash
chmod 755 myservice.sh
~~~


##  How to create custom service:

To create a custom script, you need to go into the following directory and run the following commands. Here we are going to use the multi user agent folder, that will be loaded at the boot time.

```
cd /etc/systemd/system/multi-user.target.wants/
vim testservice.service
```
Now paste the following code inside the testservice.service file.

~~~bash
[Unit]
Description = Test Service by techken.in
After = network.target
 
[Service]
ExecStart = /root/scripts/myservice.sh
 
[Install]
WantedBy = multi-user.target

~~~
Here, the script will be loaded after the network.target and it will be used by the multi-user.target.

Now enable the service to run at the startup using the command below.

```
systemctl enable testservice.service

```
After rebooting you can check the /tmp for the file “test-service-file”.

Now you have successfully created a custom service in the CentOS 7.
