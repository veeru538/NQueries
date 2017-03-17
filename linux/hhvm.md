# HHVM installation with Netdata on Ubuntu 16.04

## Install Nginx:
    sudo apt-get update
    sudo apt-get install -y nginx unzip
    sudo systemctl start nginx
    
## Install and Configure MariaDB:
    sudo apt-get install -y mariadb-client mariadb-server
    sudo systemctl start mysql
    sudo mysql_secure_installation
    sudo mysql -u root -p
    
~~~yaml    
        create database wpdatabase;
        grant all privileges on wpdatabase.* to wordpressuser@localhost identified by 'yourdbpassword';
        flush privileges;
        
~~~
## Install and Configure HHVM:
    sudo apt-get install software-properties-common
    sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0x5a16e7281be7a449 
    sudo add-apt-repository "deb http://dl.hhvm.com/ubuntu $(lsb_release -sc) main"
    sudo apt-get update
    sudo apt-get install -y hhvm
    
    
   Once the installation is complete, configure Nginx web server to use HHVM.

    sudo /usr/share/hhvm/install_fastcgi.sh

   Run this command to start HHVM automatically on system boot up.
   
    sudo update-rc.d hhvm defaults

   We’ll use HHVM as a PHP alternative.

    sudo /usr/bin/update-alternatives --install /usr/bin/php php /usr/bin/hhvm 60

  Now start HHVM.
    
    sudo systemctl start hhvm
    php -v
    
### Setup WordPress:
    cd /tmp
    wget https://wordpress.org/latest.zip
    unzip latest.zip
    sudo mv wordpress/* /var/www/html/

  Set the user and group permissions.
  
    sudo chown -R www-data:www-data /var/www/html/

  Edit the default Naginx virtual host file "/etc/nginx/sites-available/default"

    sudo vi /etc/nginx/sites-available/default
  
   Add index.php to index line.
   
 * From:
 
~~~yml     
     index index.html index.htm index.nginx-debian.html;
~~~    

 * To:
 
~~~yml    
    index index.php index.html index.htm index.nginx-debian.html;
~~~
  If you get minus 404 eroor
  
 * From:
 
~~~yml
      location / {
          try_files $uri $uri/ =404;
      }
~~~

* To:

~~~yml
    location / {
        index index.php index.html index.htm;
        try_files $uri $uri/ /index.php?$args;
    }
    
    location ~ \.php$ {
       try_files $uri /index.php;
       include fastcgi_params;
       fastcgi_pass 127.0.0.1:9000;
    }

    location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
    access_log off; log_not_found off; expires max;
    }

~~~
 
To test naginx configuration:

    nginx -t 
    sudo rm -rf /var/www/html/index.nginx-debian.html
    sudo systemctl restart nginx
    sudo systemctl restart hhvm
    
    
## Netdata – Real Time Performance Monitoring Tool for Linux:
To Install Required packages:

    sudo apt-get install zlib1g-dev gcc make git autoconf autogen automake pkg-config
    
Installing Netdata:

Run the following command to clone the netdata git.
~~~
    git clone https://github.com/firehol/netdata.git --depth=1
    cd netdata    
    sudo ./netdata-installer.sh
~~~

NetData is installed and is running.

Hit http://ip:19999/ from your browser.

To stop netdata, just kill it, with:

  killall netdata

To start it, just run it:

  /usr/sbin/netdata

If you want access netdata and your wordpress site both on nginx server add below  lines in nginx server configuration file 

~~~yml

location /netdata/ {
    rewrite ^/netdata(/.*)$ $1 break;
    proxy_pass  http://localhost:19999/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_redirect    off;
}    

~~~
  

INFORMATION:

I see you have kernel memory de-duper (called Kernel Same-page Merging,or KSM) available, but it is not currently enabled.
To enable it run:

~~~yml

  echo 1 >/sys/kernel/mm/ksm/run
  echo 1000 >/sys/kernel/mm/ksm/sleep_millisecs

~~~

If you enable it, you will save 20-60% of netdata memory.

Uninstall script generated:
   
    ./netdata-uninstaller.sh

    

    


    
    
