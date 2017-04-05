[**Linux**](../Linux.md)

# LAMP installation  on FreeBSD

To install Apache 2.4 using pkg, use this command:

      sudo pkg install apache24

To enable Apache as a service, add   `apache24_enable="YES"`   to the  `/etc/rc.conf`  file. or  We will use this sysrc command to do just that:

      sudo sysrc apache24_enable=yes
