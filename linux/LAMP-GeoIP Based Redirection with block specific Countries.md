# To install LAMP-GeoIP Based Redirection with block specific Countries
~~~/bin/bash
sudo apt-get update -y
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:ondrej/apache2
sudo apt-get update
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
a2enmod http2
sudo service apache2 restart
sudo apache2ctl configtest
sudo service apache2 restart

sudo mysql_secure_installation
sudo systemctl restart mysql

apache2ctl -M | grep -E "geoip|rewrite"



# Install Required Packages
apt update
apt upgrade
apt install geoip-bin geoip-database libapache2-mod-geoip libgeoip1


# To download GeoIp Data Set

cd /usr/share/GeoIP/
ls
rm GeoIP.dat
wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz
gunzip GeoIP.dat.gz
wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
gunzip GeoLiteCity.dat.gz
wget http://download.maxmind.com/download/geoip/database/asnum/GeoIPASNum.dat.gz
gunzip GeoIPASNum.dat.gz

a2enmod geoip
apache2ctl -M | grep -E "geoip|rewrite"
~~~

### Need To update Below Content  to Enable GeoIp based data
```
nano /etc/apache2/mods-enabled/geoip.conf
<IfModule mod_geoip.c>
  GeoIPEnable On
  GeoIPDBFile /usr/share/GeoIP/GeoIP.dat
  GeoIPDBFile /usr/share/GeoIP/GeoLiteCity.dat
  GeoIPDBFile /usr/share/GeoIP/GeoIPASNum.dat
</IfModule>

```

~~~/bin/bash
apachectl -t
sudo systemctl restart apache2.service

cd /var/www/html ;
wget https://gist.githubusercontent.com/AbhishekGhosh/b1b296f7ed6e1008f86f373688fba2e4/raw/a3bccdac0723df41dcd72a20737fda96042b0fed/geoip.php
~~~

### Need to update apache config file Directory entry
~~~
cat /etc/apache2/sites-available/000-default.conf 
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html
        <Directory />
                Options FollowSymLinks
                AllowOverride all
        </Directory>
        <Directory /var/www/>
                Options FollowSymLinks
                AllowOverride all
                Order allow,deny
                allow from all
        </Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

~~~

```

cat .htaccess 
RewriteEngine on
RewriteCond %{ENV:GEOIP_COUNTRY_CODE} ^IN$
#RewriteCond %{ENV:GEOIP_COUNTRY_CODE} ^(CN|IN)$
RewriteRule ^(.*)$ http://yourdomain.in/$1 [L]
```




# If you want to Block Specific Group  country you can specify below configuration in geoip.conf file or .htaccess file without location directoy you can added below content

~~~
nano /etc/apache2/mods-enabled/geoip.conf
<Location />
SetEnvIf GEOIP_COUNTRY_CODE CN BlockCountry
SetEnvIf GEOIP_COUNTRY_CODE EU BlockCountry
Deny from env=BlockCountry
</Location>


or 

cat .htaccess 
SetEnvIf GEOIP_COUNTRY_CODE CN BlockCountry
SetEnvIf GEOIP_COUNTRY_CODE EU BlockCountry
Deny from env=BlockCountry
~~~


# ISO 3166 Country Codes:
## List of ISO 3166 Country Codes to be used with GeoIP apache module:
```
A1 - "Anonymous Proxy"
A2 - "Satellite Provider"
O1 - "Other Country"
AD - "Andorra"
AE - "United Arab Emirates"
AF - "Afghanistan"
AG - "Antigua and Barbuda"
AI - "Anguilla"
AL - "Albania"
AM - "Armenia"
AO - "Angola"
AP - "Asia/Pacific Region"
AQ - "Antarctica"
AR - "Argentina"
AS - "American Samoa"
AT - "Austria"
AU - "Australia"
AW - "Aruba"
AX - "Aland Islands"
AZ - "Azerbaijan"
BA - "Bosnia and Herzegovina"
BB - "Barbados"
BD - "Bangladesh"
BE - "Belgium"
BF - "Burkina Faso"
BG - "Bulgaria"
BH - "Bahrain"
BI - "Burundi"
BJ - "Benin"
BL - "Saint Bartelemey"
BM - "Bermuda"
BN - "Brunei Darussalam"
BO - "Bolivia"
BQ - "Bonaire - Saint Eustatius and Saba"
BR - "Brazil"
BS - "Bahamas"
BT - "Bhutan"
BV - "Bouvet Island"
BW - "Botswana"
BY - "Belarus"
BZ - "Belize"
CA - "Canada"
CC - "Cocos (Keeling) Islands"
CD - "Congo - The Democratic Republic of the"
CF - "Central African Republic"
CG - "Congo"
CH - "Switzerland"
CI - "Cote d'Ivoire"
CK - "Cook Islands"
CL - "Chile"
CM - "Cameroon"
CN - "China"
CO - "Colombia"
CR - "Costa Rica"
CU - "Cuba"
CV - "Cape Verde"
CW - "Curacao"
CX - "Christmas Island"
CY - "Cyprus"
CZ - "Czech Republic"
DE - "Germany"
DJ - "Djibouti"
DK - "Denmark"
DM - "Dominica"
DO - "Dominican Republic"
DZ - "Algeria"
EC - "Ecuador"
EE - "Estonia"
EG - "Egypt"
EH - "Western Sahara"
ER - "Eritrea"
ES - "Spain"
ET - "Ethiopia"
EU - "Europe"
FI - "Finland"
FJ - "Fiji"
FK - "Falkland Islands (Malvinas)"
FM - "Micronesia - Federated States of"
FO - "Faroe Islands"
FR - "France"
GA - "Gabon"
GB - "United Kingdom"
GD - "Grenada"
GE - "Georgia"
GF - "French Guiana"
GG - "Guernsey"
GH - "Ghana"
GI - "Gibraltar"
GL - "Greenland"
GM - "Gambia"
GN - "Guinea"
GP - "Guadeloupe"
GQ - "Equatorial Guinea"
GR - "Greece"
GS - "South Georgia and the South Sandwich Islands"
GT - "Guatemala"
GU - "Guam"
GW - "Guinea-Bissau"
GY - "Guyana"
HK - "Hong Kong"
HM - "Heard Island and McDonald Islands"
HN - "Honduras"
HR - "Croatia"
HT - "Haiti"
HU - "Hungary"
ID - "Indonesia"
IE - "Ireland"
IL - "Israel"
IM - "Isle of Man"
IN - "India"
IO - "British Indian Ocean Territory"
IQ - "Iraq"
IR - "Iran - Islamic Republic of"
IS - "Iceland"
IT - "Italy"
JE - "Jersey"
JM - "Jamaica"
JO - "Jordan"
JP - "Japan"
KE - "Kenya"
KG - "Kyrgyzstan"
KH - "Cambodia"
KI - "Kiribati"
KM - "Comoros"
KN - "Saint Kitts and Nevis"
KP - "Korea - Democratic People's Republic of"
KR - "Korea - Republic of"
KW - "Kuwait"
KY - "Cayman Islands"
KZ - "Kazakhstan"
LA - "Lao People's Democratic Republic"
LB - "Lebanon"
LC - "Saint Lucia"
LI - "Liechtenstein"
LK - "Sri Lanka"
LR - "Liberia"
LS - "Lesotho"
LT - "Lithuania"
LU - "Luxembourg"
LV - "Latvia"
LY - "Libyan Arab Jamahiriya"
MA - "Morocco"
MC - "Monaco"
MD - "Moldova - Republic of"
ME - "Montenegro"
MF - "Saint Martin"
MG - "Madagascar"
MH - "Marshall Islands"
MK - "Macedonia"
ML - "Mali"
MM - "Myanmar"
MN - "Mongolia"
MO - "Macao"
MP - "Northern Mariana Islands"
MQ - "Martinique"
MR - "Mauritania"
MS - "Montserrat"
MT - "Malta"
MU - "Mauritius"
MV - "Maldives"
MW - "Malawi"
MX - "Mexico"
MY - "Malaysia"
MZ - "Mozambique"
NA - "Namibia"
NC - "New Caledonia"
NE - "Niger"
NF - "Norfolk Island"
NG - "Nigeria"
NI - "Nicaragua"
NL - "Netherlands"
NO - "Norway"
NP - "Nepal"
NR - "Nauru"
NU - "Niue"
NZ - "New Zealand"
OM - "Oman"
PA - "Panama"
PE - "Peru"
PF - "French Polynesia"
PG - "Papua New Guinea"
PH - "Philippines"
PK - "Pakistan"
PL - "Poland"
PM - "Saint Pierre and Miquelon"
PN - "Pitcairn"
PR - "Puerto Rico"
PS - "Palestinian Territory"
PT - "Portugal"
PW - "Palau"
PY - "Paraguay"
QA - "Qatar"
RE - "Reunion"
RO - "Romania"
RS - "Serbia"
RU - "Russian Federation"
RW - "Rwanda"
SA - "Saudi Arabia"
SB - "Solomon Islands"
SC - "Seychelles"
SD - "Sudan"
SE - "Sweden"
SG - "Singapore"
SH - "Saint Helena"
SI - "Slovenia"
SJ - "Svalbard and Jan Mayen"
SK - "Slovakia"
SL - "Sierra Leone"
SM - "San Marino"
SN - "Senegal"
SO - "Somalia"
SR - "Suriname"
SS - "South Sudan"
ST - "Sao Tome and Principe"
SV - "El Salvador"
SX - "Sint Maarten"
SY - "Syrian Arab Republic"
SZ - "Swaziland"
TC - "Turks and Caicos Islands"
TD - "Chad"
TF - "French Southern Territories"
TG - "Togo"
TH - "Thailand"
TJ - "Tajikistan"
TK - "Tokelau"
TL - "Timor-Leste"
TM - "Turkmenistan"
TN - "Tunisia"
TO - "Tonga"
TR - "Turkey"
TT - "Trinidad and Tobago"
TV - "Tuvalu"
TW - "Taiwan"
TZ - "Tanzania - United Republic of"
UA - "Ukraine"
UG - "Uganda"
UM - "United States Minor Outlying Islands"
US - "United States"
UY - "Uruguay"
UZ - "Uzbekistan"
VA - "Holy See (Vatican City State)"
VC - "Saint Vincent and the Grenadines"
VE - "Venezuela"
VG - "Virgin Islands - British"
VI - "Virgin Islands - U.S."
VN - "Vietnam"
VU - "Vanuatu"
WF - "Wallis and Futuna"
WS - "Samoa"
YE - "Yemen"
YT - "Mayotte"
ZA - "South Africa"
ZM - "Zambia"
ZW - "Zimbabwe"
```
