# Lamp Instalation Script for Ubuntu16.04
~~~yml
#!/bin/bash
sudo apt-get update -y
sudo apt-get install apache2 -y 
sudo apache2ctl configtest
sudo apt-get install curl -y 
sudo apt install mysql-server mysql-client -y 
sudo apt-get install php libapache2-mod-php php-mcrypt php-mysql -y 
sudo systemctl restart apache2
sudo apt-get install  php-common php-gd php-mysql php-mcrypt php-curl php-intl  php-mbstring php-zip php-bcmath php-iconv    -y 
sudo apt-get install php-xml php-mbstring php-gettext -y 
sudo apt-get install php-apcu -y

cat >  /etc/apache2/mods-enabled/dir.conf  <<EOF
<IfModule mod_dir.c>
        DirectoryIndex index.php  index.html index.cgi index.pl index.php index.xhtml index.htm
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
EOF

## Remove Apache server information from headers. 
sed -i 's/ServerTokens .*/ServerTokens Prod/' /etc/apache2/conf-available/security.conf
sed -i 's/ServerSignature .*/ServerSignature Off/' /etc/apache2/conf-available/security.conf

echo "Header unset ETag" >> /etc/apache2/conf-available/security.conf
echo "Header always unset X-Powered-By" >> /etc/apache2/conf-available/security.conf
echo "FileETag None" >> /etc/apache2/conf-available/security.conf


a2enmod headers
a2enmod rewrite
a2enmod expires
a2enmod deflate
sudo service apache2 restart
sudo apache2ctl configtest
sudo service apache2 restart


sudo mysql_secure_installation
sudo systemctl restart mysql
#sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf    # This is mysql5.7 configuration file location
sudo systemctl restart mysql



#SWAP partition
dd if=/dev/zero of=/swapfile count=2048 bs=1M
ls -lh /swapfile
chmod 600 /swapfile
ls -lh /swapfile
mkswap /swapfile
swapon /swapfile
swapon -s
echo "/swapfile   none    swap    sw    0   0" >> /etc/fstab
echo  1  > /proc/sys/net/ipv4/icmp_echo_ignore_all

cat >>  /etc/sysctl.conf  <<EOF

# IP Spoofing protection
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Ignore ICMP broadcast requests
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Disable source packet routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0 
net.ipv4.conf.default.accept_source_route = 0
net.ipv6.conf.default.accept_source_route = 0

# Ignore send redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Block SYN attacks
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 5

# Log Martians
net.ipv4.conf.all.log_martians = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Ignore ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0 
net.ipv6.conf.default.accept_redirects = 0

# Ignore Directed pings
net.ipv4.icmp_echo_ignore_all = 1

EOF

sudo sysctl -p

~~~
