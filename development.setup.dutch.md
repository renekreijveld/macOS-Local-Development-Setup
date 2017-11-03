# Disclaimer

Deze setup werkt bij mij probleemloos. Ik ben zeker geen expert in Apache, PHP en MySQL dus als er iets mis gaat, kijk dan vooral bij de bronnen die ik gebruikt heb.

# Benodigdheden

- macOS 10.12 of hoger
- <a href="https://developer.apple.com/xcode/" target="_blank">XCode</a> geïnstalleerd

# Gebruikte bronnen

<a href="https://getgrav.org/blog/macos-sierra-apache-multiple-php-versions" target="_blank">macOS 10.13 High Sierra Apache Setup: Multiple PHP Versions</a>

<a href="https://coolestguidesontheplanet.com/get-apache-mysql-php-phpmyadmin-working-osx-10-10-yosemite/#phpmyadmin" target="_blank">Get Apache, MySQL, PHP and phpMyAdmin working on OSX 10.10 Yosemite</a>

# Installeren XCode Command Line Tools

```
$ xcode-select --install
```

# Homebrew installatie

```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Controle installatie Homebrew

```
$ brew --version
Homebrew 1.3.6
Homebrew/homebrew-core (git revision 94caa; last commit 2017-10-22)
```

### Extra Brew Taps toevoegen

```
$ brew tap homebrew/php
```

### Bestaande Apache stoppen en Homebrew versie installaren

```
$ sudo apachectl stop
$ sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
$ brew install httpd
```

Je kunt Apache instellen zodat die elke start bij reboot van je machine.

```
$ sudo brew services start httpd
```

Je Apache ook handmatig starten.

```
$ sudo apachectl start
```

Test Apache door in browser te gaan naar: http://localhost:8080

### Apache configuratie aanpassen

```
$ open -e /usr/local/etc/httpd/httpd.conf
```

Vervang:

```
Listen 8080
```

Door:

```
Listen 80
```

Vervang:

```
DocumentRoot "/usr/local/var/www"
```

Door:

```
DocumentRoot /Users/your_user/Development/Sites (dus ZONDER quotes)
```

(vul bij 'your_user' jouw eigen gebruikernaam in)

Vervang:

```
<Directory "/usr/local/var/www">
```

Door:

```
<Directory /Users/your_user/Development/Sites> (dus ZONDER quotes)
```

(vul bij 'your_user' jouw eigen gebruikernaam in)

In hetzelfde <Directory blok:

Vervang:

```
AllowOverride None
```

Door:

```
AllowOverride All
```

Zoek naar:

```
#LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so
```

En vervang door:

```
LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so (dus # weghalen)
```

### User & Group aanpassen

Vervang:

```
User _www
Group _www
```

Door:

```
User your_user
Group staff
```

(vul bij 'your_user' jouw eigen gebruikernaam in)

### Servername instellen

Vervang:

```
#ServerName www.example.com:8080
```

Door:

```
ServerName localhost
```

Sla het bestand /usr/local/etc/httpd/httpd.conf op.

### Standaard index.html aanmaken

```
$ mkdir -p ~/Development/Sites
$ echo "<h1>My User Web Root</h1>" > ~/Development/Sites/index.html
```

### Apache herstarten

```
$ sudo apachectl -k restart
```

Ga in je browser naar http://localhost, daar moet dan My User Web Root verschijnen.

# PHP installeren

### PHP 5.3 installeren

```
$ brew install php53 --with-httpd
```

### PHP 5.6 installeren

```
$ brew unlink php53
$ brew install php56 --with-httpd
```

### PHP 7.0 installeren

```
$ brew unlink php56
$ brew install php70 --with-httpd
```

### PHP 7.1 installeren

```
$ brew unlink php70
$ brew install php71 --with-httpd
```

### PHP 7.2 installeren

```
$ brew unlink php71
$ brew install php72 --with-httpd
```

### Terug wisselen naar PHP 5.6

```
$ brew unlink php72
$ brew link php56
```

# PHP in Apache setup

### Apache configuratie aanpassen

```
$ open -e /usr/local/etc/httpd/httpd.conf
```

Vervang:

```
LoadModule php5_module        /usr/local/Cellar/php53/5.3.29_8/libexec/apache2/libphp5.so
LoadModule php5_module        /usr/local/Cellar/php56/5.6.32_8/libexec/apache2/libphp5.so
LoadModule php7_module        /usr/local/Cellar/php70/7.0.25_17/libexec/apache2/libphp7.so
LoadModule php7_module        /usr/local/Cellar/php71/7.1.11_22/libexec/apache2/libphp7.so
LoadModule php7_module        /usr/local/Cellar/php72/7.2.0RC5_8/libexec/apache2/libphp7.so
```

Door:

```
#LoadModule php5_module    /usr/local/opt/php53/libexec/apache2/libphp5.so
LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
#LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so
#LoadModule php7_module    /usr/local/opt/php71/libexec/apache2/libphp7.so
#LoadModule php7_module    /usr/local/opt/php72/libexec/apache2/libphp7.so
```

Vervang:

```
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

Door:

```
<IfModule dir_module>
    DirectoryIndex kick.php index.php index.html
</IfModule>

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```

Sla het bestand op en herstart apache:

```
$ sudo apachectl -k stop
$ sudo apachectl start
```

Maak een bestand info.php in je ~/Development/Sites map met deze inhoud:

```php
<?php phpinfo();
```

Test de werking van PHP door in je browser te gaan naar http://localhost/info.php

# PHP Switcher script installeren

Om gemakkelijk tussen PHP versies te kunnen wisselen installeren we een PHP switch script.

```
$ curl -L https://gist.github.com/w00fz/142b6b19750ea6979137b963df959d11/raw > /usr/local/bin/sphp
$ chmod +x /usr/local/bin/sphp
```

### Apache configuratie aanpassen

```
$ open -e /usr/local/etc/httpd/httpd.conf
```

Vervang:

```
#LoadModule php5_module    /usr/local/opt/php53/libexec/apache2/libphp5.so
LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
#LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so
#LoadModule php7_module    /usr/local/opt/php71/libexec/apache2/libphp7.so
#LoadModule php7_module    /usr/local/opt/php72/libexec/apache2/libphp7.so
```

Door:

```
#Brew PHP LoadModule for `sphp` switcher23.1605
LoadModule php5_module /usr/local/lib/libphp5.so
#LoadModule php7_module /usr/local/lib/libphp7.so
```

Sla het bestand op. Test het switcher script:

```
$ sphp 70
```

Ververs de pagina <a href="http://localhost/info.php" target="_blank">http://localhost/info.php</a> in je browser.

# PHP.ini aanpassen

Om webapplicaties goed te laten werken passen we een aantal zaken aan in de php.ini's.
De volgende aanpassingen moeten worden doorgevoerd. Zoek daarvoor de originele instelling op, bewaar die door de regel te kopieren en te voorzien van een ; aan het begin van de regel en plaats dan de nieuwe waarde.

Voor display_errors moet je mogelijk een uitzondering maken en die op 'On' laten staan, maar dat is een persoonlijke voorkeur.

```
output_buffering = Off
max_execution_time = 180
max_input_time = 180
memory_limit = 256M
display_errors = Off
post_max_size = 50M
upload_max_filesize = 50M
date.timezone = Europe/Amsterdam
```

Aanpassen php.ini PHP 5.3:

```
$ open -e /usr/local/etc/php/5.3/php.ini
```

Aanpassen php.ini PHP 5.6:

```
$ open -e /usr/local/etc/php/5.6/php.ini
```

Aanpassen php.ini PHP 7.0:

```
$ open -e /usr/local/etc/php/7.0/php.ini
```

Aanpassen php.ini PHP 7.1:

```
$ open -e /usr/local/etc/php/7.1/php.ini
```

Aanpassen php.ini PHP 7.2:

```
$ open -e /usr/local/etc/php/7.2/php.ini
```

Herstart Apache na de php.ini aanpassingen:

```
$ sudo apachectl -k restart
```

# MySQL installeren

```
$ brew update
$ brew install mysql
```

### Start MySQL

```
$ brew services start mysql
```

### MySQL beveiligen

```
$ /usr/local/bin/mysql_secure_installation
```

Antwoorden:

```
Would you like to setup VALIDATE PASSWORD plugin? Press y|Y for Yes, any other key for No: Enter (= no)
Please set the password for root here. New password: root
Re-enter new password: root
Remove anonymous users? y
Disallow root login remotely? y
Remove test database and access to it? y
Reload privilege tables now? y
```

# phpMyAdmin installeren

Download de Engelse phpMyAdmin via http://www.phpmyadmin.net/home_page/downloads.php

Unzip het bestand en verplaats de map naar de submap 'phpmyadmin' onder Sites, dus naar ~/Development/Sites/phpmyadmin

### Setup uitvoeren in de browser

Open in je browser http://localhost/phpmyadmin/setup/

- Klik op New Server
- Tabblad Basic settings: niets aanpassen
- Tabblad Authentication:
    - bij Authentication type kies: config
    - bij Password for config auth invullen: root
    - klik op Apply, je komt terug in Overview
- Klik op download en sla het bestand config.inc.php op in de map ~/Development/Sites/phpmyadmin

Ga dan in je browser naar http://localhost/phpmyadmin. Je moet nu phpMyAdmin zien met in de linker kolom de databases.

# Apache Virtual Hosts

### Aanpassen Apache configuratie

```
$ open -e /usr/local/etc/httpd/httpd.conf
```

Vervang:

```
#LoadModule vhost_alias_module lib/httpd/modules/mod_vhost_alias.so
```

Door:

```
LoadModule vhost_alias_module lib/httpd/modules/mod_vhost_alias.so
```

Vervang:

```
#Include /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

Door:

```
Include /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

Aanpassen httpd-vhosts.conf:

```
$ open -e /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

Verwijder alle bestaande regels onder het commentaar blok en plaats alleen de volgende regels onder het commentaarblok:

```
<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Development/Sites"
    ServerName localhost
</VirtualHost>

<Directory "/Users/your_user/Development/Sites">
    Allow From All
    AllowOverride All
    Options +Indexes
    Require all granted
</Directory>

<Virtualhost *:80>
    VirtualDocumentRoot "/Users/your_user/Development/Sites/%1"
    ServerAlias *.dev
    UseCanonicalName Off
</Virtualhost>
```

(vul bij 'your_user' jouw eigen gebruikernaam in)

# Dnsmasq installeren

Via bovenstaande setup hebben we het nu heel eenvoudig gemaakt om een nieuwe virtuele host toe te voegen.
Door een submap te maken in de map ~/Development/Sites/ bijvoorbeeld 'testsite', zal de direct als virtuele host te benaderen zijn via de domeinnaam 'testsite.dev'.

Deze domeinnaam wordt niet automatisch geresolved op je computer en daarvoor installeren we de tool Dnsmasq.

```
$ brew install dnsmasq
```

Instellen *.dev hosts:

```
$ echo 'address=/.dev/127.0.0.1' > /usr/local/etc/dnsmasq.conf
```

Dnsmasq starten en instellen dat bij elke reboot dnsmasq wordt gestart:

```
$ sudo brew services start dnsmasq
```

Toevoegen aan resolvers:

```
$ sudo mkdir -v /etc/resolver
$ sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/dev'
```

Testen door het pingen naar een onbekende .dev naam. Er moet een reply komen van 127.0.0.1:

```
$ ping nieuwewebsite.dev
```

Herstart apache:

```
$ sudo apachectl -k restart
```

# APC Cache en XDebug installeren

Om PHP sneller te laten werken installeren we Zend OPcache en APCu Cache.

```
$ sphp 53
$ brew install php53-opcache php53-apcu --build-from-source
$ brew install php53-xdebug --build-from-source
```

Herhaal dit proces voor de andere PHP versies.

```
$ sphp 56
$ brew install php56-opcache php56-apcu --build-from-source
$ brew install php56-xdebug --build-from-source
```

```
$ sphp 70
$ brew install php70-opcache php70-apcu --build-from-source
$ brew install php70-xdebug --build-from-source
```

```
$ sphp 71
$ brew install php71-opcache php71-apcu --build-from-source
$ brew install php71-xdebug --build-from-source
```

```
$ sphp 72
$ brew install php72-opcache php72-apcu --build-from-source
```

XDebug is nog niet beschikbaar voor PHP 7.2.

Herstart apache:

```
$ sudo apachectl -k restart
```

XDebug enable/disable script installeren:

```
$ brew install xdebug-osx
```

De standaard configuraties werken wellicht niet goed genoeg. We passen daarom de configuraties aan.

PHP 5.3:

```
$ open -e /usr/local/etc/php/5.3/conf.d/ext-xdebug.ini
```

Vervang de inhoud door deze inhoud:

```
[xdebug]
zend_extension="/usr/local/opt/php53-xdebug/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

PHP 5.6:

```
$ open -e /usr/local/etc/php/5.6/conf.d/ext-xdebug.ini
```

Vervang de inhoud door deze inhoud:

```
[xdebug]
zend_extension="/usr/local/opt/php56-xdebug/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

PHP 7.0:

```
$ open -e /usr/local/etc/php/7.0/conf.d/ext-xdebug.ini
```

Vervang de inhoud door deze inhoud:

```
[xdebug]
zend_extension="/usr/local/opt/php70-xdebug/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

PHP 7.1:

```
$ open -e /usr/local/etc/php/7.1/conf.d/ext-xdebug.ini
```

Vervang de inhoud door deze inhoud:

```
[xdebug]
zend_extension="/usr/local/opt/php71-xdebug/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

Herstart Apache:

```
$ sudo apachectl -k restart
```

Je kunt nu eenvoudig XDebug aan- of uitzetten.
XDebug aanzetten voor de actieve PHP versie:

```
$ xdebug-toggle on
```

XDebug uitzetten voor de actieve PHP versie:

```
$ xdebug-toggle off
```

# Startdevelopment en Stopdevelopment

Persoonlijk wil ik niet dat bij elke reboot Apache en MySQL automatisch worden gestart.
Ik gebruik voor het starten en stoppen van Apache en MySQL twee eenvoudige scripts:

```
$ touch /usr/local/bin/startdevelopment
$ open -e /usr/local/bin/startdevelopment
```

Plaats daarin de volgende code:

```
#!/bin/bash

# Start apache
sudo apachectl start

# Start mysql
brew services start mysql
```

Sla dit bestand op.

```
$ touch /usr/local/bin/stopdevelopment
$ open -e /usr/local/bin/stopdevelopment
```

Plaats daarin de volgende code:

```
#!/bin/bash

# Start mysql
brew services stop mysql

# Start apache
sudo apachectl stop
```

Sla dit bestand op.

Rechten aanpassen:

```
$ chmod +x /usr/local/bin/startdevelopment /usr/local/bin/stopdevelopment
```

