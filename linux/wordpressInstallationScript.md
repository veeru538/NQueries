
# WordPress instllation Sctipt 
```
#!/bin/bash
echo “Database Name: ”
read -e dbname
echo “Database User: ”
read -e dbuser
echo “Database Password: ”
read -s dbpass
echo “run install? (y/n)”
read -e run
if [ "$run" == n ] ; then
exit
else
#download wordpress
cd /tmp
curl -O http://wordpress.org/latest.tar.gz
#unzip wordpress
tar zxvf latest.tar.gz --strip 1 -C  /var/www/html
cd /var/www/html
#create wp config
cp wp-config-sample.php wp-config.php
#set database details with perl find and replace
sed -e 's/database_name_here/$dbname/g' wp-config.php
sed -e 's/username_here/$dbuser/g' wp-config.php
sed 's/password_here/$dbpass/g' wp-config.php
#create uploads folder and set permissions
mkdir wp-content/uploads
chmod 777 wp-content/uploads
find . -type d -exec chmod 755 {} \;
find . -type f -exec chmod 644 {} \;
#remove bash script
rm wp.sh
fi
```
