[**Linux**](../Linux.md)

# LAMP installation  on FreeBSD

To install Apache 2.4 using pkg, use this command:

      sudo pkg install apache24

To enable Apache as a service, add   `apache24_enable="YES"`   to the  `/etc/rc.conf`  file. or  We will use this sysrc command to do just that:

      sudo sysrc apache24_enable=yes

Now start Apache:

      sudo service apache24 start

To install MySQL 5.6 using pkg, use this command:

      sudo pkg install mysql56-server
      
To enable MySQL server as a service, add `mysql_enable="YES"` to the `/etc/rc.conf` file. This sysrc command will do just that:

      sudo sysrc mysql_enable=yes

Now start the MySQL server:

      sudo service mysql-server start

      sudo mysql_secure_installation
      
To install PHP 5.6 with pkg, run this command:

      sudo pkg install mod_php56 php56-mysql php56-mysqli

Now copy the sample PHP configuration file into place with this command:

      sudo cp /usr/local/etc/php.ini-production /usr/local/etc/php.ini

Now run the rehash command to regenerate the system's cached information about your installed executable files:

      rehash

Install PHP Modules:
      
      sudo pkg search php56

To get more information about each module does, you can either search the internet, or you can look at the long description of the package by typing:

      sudo pkg search -f package_name

Configure Apache to Use PHP Module:
Before Apache will process PHP pages, we must configure it to use mod_php.

Open the Apache configuration file and edit it :      

      sudo nano /usr/local/etc/apache24/httpd.conf      

```
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```
    
To

``` 
<IfModule dir_module>
    DirectoryIndex index.php index.html
    <FilesMatch "\.php$">
        SetHandler application/x-httpd-php
    </FilesMatch>
    <FilesMatch "\.phps$">
        SetHandler application/x-httpd-php-source
    </FilesMatch>
 </IfModule>
```
Test PHP Processing:
            
      sudo vi /usr/local/www/apache24/data/info.php
      
~~~php

      <?php phpinfo(); ?>
~~~
      
## How To Configure a Simple IPFW Firewall:

Configuring the Basic Firewall 
Almost all of our configuration will take place in the `/etc/rc.conf` file
         
         sudo vi /etc/rc.conf

To add below rules:
```
firewall_enable="YES"
firewall_quiet="YES"
firewall_type="workstation"
firewall_myservices="22 80"
firewall_allowservices="any"
firewall_logdeny="YES"
```
To Starting the Firewall:
 
      sudo service ipfw start

## how to change Shell in FreeBSD:
check default shell:
      
      env | grep SHELL
      
To cange shell csh to bash 
      
       chsh -s /usr/local/bin/bash
