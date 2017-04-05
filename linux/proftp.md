[**Linux**](../Linux.md)

# ProFTPd with TLS support Installation and Configuration  on Ubuntu 16.04

Before  you can start setup swich shell root user mode 
~~~yaml    
  sudo -s
~~~

## Install ProFTPd and OpenSSL:
~~~yaml    
  apt-get -y install proftpd openssl
~~~
It will ask for some question about ProFTPD, select standalone and press Ok

![image1](/images/proftp1.png)

## We can check the ProFTPD version as follows:
~~~yaml 
root@dev1:~# proftpd -v
ProFTPD Version 1.3.4a
~~~
For security reasons, you should add the following lines to /etc/proftpd/proftpd.conf:

nano /etc/proftpd/proftpd.conf
~~~yaml 
DefaultRoot         ~
ServerIdent on "FTP Server ready."
~~~
The first option enables chrooting of FTP users into their home directory and the second option enables a ServerIdent message that does not contain any information about the used FTP server software, version or OS so that a potential attacker don't gets these details on the silver plate.

## Create the SSL Certificate for TLS:

In order to use TLS, we must create an SSL certificate. I create it in /etc/proftpd/ssl, therefore I create that directory first:
~~~yaml 
mkdir /etc/proftpd/ssl -p
~~~
Afterward, we can generate the SSL certificate as follows:
~~~yaml 
openssl req -new -x509 -days 365 -nodes -out /etc/proftpd/ssl/proftpd.cert.pem -keyout /etc/proftpd/ssl/proftpd.key.pem
~~~

~~~yaml 
Country Name (2 letter code) [AU]: <-- Enter your Country Name (e.g., "IN").
State or Province Name (full name) [Some-State]:<-- Enter your State or Province Name.
Locality Name (eg, city) []:<-- Enter your City.
Organization Name (eg, company) [Internet Widgits Pty Ltd]:<-- Enter your Organization Name (e.g., the name of your company).
Organizational Unit Name (eg, section) []:<-- Enter your Organizational Unit Name (e.g. "IT Department").
Common Name (eg, YOUR name) []:<-- Enter the Fully Qualified Domain Name of the system (e.g. "dev1.example.com").
Email Address []:<-- Enter your Email Address.
~~~

 and secure the generated certificate files.
~~~yaml 
chmod 600 /etc/proftpd/ssl/proftpd.*
~~~
## Enable TLS in ProFTPd:
In order to enable TLS in ProFTPd, open /etc/proftpd/proftpd.conf...

Uncomment the Include /etc/proftpd/tls.conf line in  /etc/proftpd/proftpd.conf file:
~~~yml
Include /etc/proftpd/tls.conf
~~~

Then open /etc/proftpd/tls.conf and make it look as follows:
~~~yml
nano /etc/proftpd/tls.conf

<IfModule mod_tls.c>
TLSEngine                  on
TLSLog                     /var/log/proftpd/tls.log
TLSProtocol TLSv1.2
TLSCipherSuite AES128+EECDH:AES128+EDH
TLSOptions                 NoCertRequest AllowClientRenegotiations
TLSRSACertificateFile      /etc/proftpd/ssl/proftpd.cert.pem
TLSRSACertificateKeyFile   /etc/proftpd/ssl/proftpd.key.pem
TLSVerifyClient            off
TLSRequired                on
RequireValidShell          no
</IfModule>
~~~

If you use TLSRequired on, then only TLS connections are allowed (this locks out any users with old FTP clients that don't have TLS support); by commenting out that line or using TLSRequired off both TLS and non-TLS connections are allowed, depending on what the FTP.

## Restart ProFTPd  service:
~~~yml
systemctl restart proftpd.service
~~~
That's it. You can now try to connect using your FTP client; however, you should configure your FTP client to use TLS (this is a must if you use TLSRequired on) - see the next chapter how to do this with FileZilla.

If you're having problems with TLS, you can take a look at the TLS log file '/var/log/proftpd/tls.log.'

 
## Add an FTP user:

The ProFTPD configuration used in thus tutorial authenticates users against the Linux system user database (/etc/passwd and /etc/shadow). 
In this step, I will add a user "tom" to be used for FTP login only.
~~~yml
useradd --shell /bin/false veeru1
~~~
Then we have to create the home directory of our user "veeru1" and change the ownership of that directory to the user and group "veeru1".
~~~yml
mkdir /home/veeru1
chown veeru1:veeru1 /home/veeru1/
~~~

This will add the user "veeru1" with the shell /bin/false. This shell ensures that he can login by FTP but not by SSH.

AnotherWay :
~~~yml
useradd --home /home/veeru1 --create-home --shell /bin/false veeru1
~~~
## To Set password FTP User:
~~~yml
passwd veeru1
~~~
