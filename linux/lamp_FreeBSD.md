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


