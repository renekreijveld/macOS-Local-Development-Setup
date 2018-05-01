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

### Wget installeren

```
$ brew install wget
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

Je Apache ook handmatig starten, stoppen en herstarten

```
$ sudo apachectl start
$ sudo apachectl stop
$ sudo apachectl -k restart
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

### PHP 5.6 installeren

```
$ brew install php@5.6
```

### PHP 7.0 installeren

```
$ brew install php@7.0
```

### PHP 7.1 installeren

```
$ brew install php@7.1
```

### PHP 7.2 installeren

```
$ brew install php@7.2
```

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

Terug naar de eerste PHP versie

```
$ brew unlink php@7.2 && brew link --force --overwrite php@5.6
```

Ik adviseer nu om alle Terminal tabs en windows de sluiten. Daarna open je een nieuwe Terminal en ga je verder met de volgende stap. Doe dit omdat er nog eens vreemd issues kunnen optreden met de PATH instellingen in de bestaande Terminals.

Snelle test of we in de juiste PHP versie zitten:

```
$ php -v
PHP 5.6.36 (cli) (built: Apr 26 2018 22:02:57) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
```

# PHP in Apache setup

### Apache configuratie aanpassen

```
$ open -e /usr/local/etc/httpd/httpd.conf
```

Zoek de regel met de mod_rewrite module:

LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so

Voeg hieronder de volgende regels toe:

```
LoadModule php5_module /usr/local/opt/php@5.6/lib/httpd/modules/libphp5.so
#LoadModule php7_module /usr/local/opt/php@7.0/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.1/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.2/lib/httpd/modules/libphp7.so
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
$ curl -L https://gist.githubusercontent.com/rhukster/f4c04f1bf59e0b74e335ee5d186a98e2/raw > /usr/local/bin/sphp
$ chmod +x /usr/local/bin/sphp
```

# Controleer je lokale pad

```
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

Als je dit niet ziet, moet het pad handmatig worden aangepast. Afhankelijk van de shell die je gebruiikt moet je aanpassingfen doen in ~/.profile, ~/.bash_profile, of ~/.zshrc. Ik ga er van uit dat je je de standaard bash shell gebruikt, dus voeg voeg deze regel toe aan je .profile (maak dit bestand als die niet bestaat) in de root van je homedirectory:

```
$ open -e ~/.profile
export PATH=/usr/local/bin:/usr/local/sbin:$PATH
```

Stop en herstart dan je Terminal.

### PHP switcher testen

Test het switcher script:

```
$ sphp 7.0
```

Ververs de pagina <a href="http://localhost/info.php" target="_blank">http://localhost/info.php</a> in je browser.

# De software updatyen

Brew makes it super easy to update PHP and the other packages you install. The first step is to update Brew so that it gets a list of available updates:

```
$ brew update
```

This will spit out a list of available updates, and any deleted formulas. To upgrade the packages simply type:

```
$ brew upgrade
```

To update all of your PHP versions you have to switch to them, and then run brew update.

```
$ sphp 5.6
Switching to php@5.6
Switching your shell
Unlinking /usr/local/Cellar/php@5.6/5.6.36... 0 symlinks removed
Unlinking /usr/local/Cellar/php@7.0/7.0.30... 0 symlinks removed
Unlinking /usr/local/Cellar/php@7.1/7.1.17... 0 symlinks removed
Unlinking /usr/local/Cellar/php/7.2.5... 24 symlinks removed
Linking /usr/local/Cellar/php@5.6/5.6.36... 25 symlinks created

If you need to have this software first in your PATH instead consider running:
  echo 'export PATH="/usr/local/opt/php@5.6/bin:$PATH"' >> ~/.bash_profile
  echo 'export PATH="/usr/local/opt/php@5.6/sbin:$PATH"' >> ~/.bash_profile
You will need sudo power from now on
Switching your apache conf
Restarting apache

PHP 5.6.36 (cli) (built: Apr 26 2018 22:02:57) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies

All done!
$ brew update
Already up-to-date.
```

Repeat these steps for PHP 7.0, 7.1 and 7.2.

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
    ServerAlias *.test
    UseCanonicalName Off
</Virtualhost>
```

(vul bij 'your_user' jouw eigen gebruikernaam in)

# Dnsmasq installeren

Via bovenstaande setup hebben we het nu heel eenvoudig gemaakt om een nieuwe virtuele host toe te voegen.
Door een submap te maken in de map ~/Development/Sites/ bijvoorbeeld 'testsite', zal de direct als virtuele host te benaderen zijn via de domeinnaam 'testsite.test'.

Deze domeinnaam wordt niet automatisch geresolved op je computer en daarvoor installeren we de tool Dnsmasq.

```
$ brew install dnsmasq
```

Instellen *.test hosts:

```
$ echo 'address=/.test/127.0.0.1' > /usr/local/etc/dnsmasq.conf
```

Dnsmasq starten en instellen dat bij elke reboot dnsmasq wordt gestart:

```
$ sudo brew services start dnsmasq
```

Toevoegen aan resolvers:

```
$ sudo mkdir -v /etc/resolver
$ sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/test'
```

Testen door het pingen naar een onbekende .test naam. Er moet een reply komen van 127.0.0.1:

```
$ ping nieuwewebsite.test
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
$ brew install php72-xdebug --build-from-source
```

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

Als je met PhpStorm werkt is het ook handig om de volgende regel nog toe te voegen:

```
xdebug.file_link_format="phpstorm://open?file=%f&line=%l"
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

Als je met PhpStorm werkt is het ook handig om de volgende regel nog toe te voegen:

```
xdebug.file_link_format="phpstorm://open?file=%f&line=%l"
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

Als je met PhpStorm werkt is het ook handig om de volgende regel nog toe te voegen:

```
xdebug.file_link_format="phpstorm://open?file=%f&line=%l"
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

Als je met PhpStorm werkt is het ook handig om de volgende regel nog toe te voegen:

```
xdebug.file_link_format="phpstorm://open?file=%f&line=%l"
```

PHP 7.2:

```
$ open -e /usr/local/etc/php/7.2/conf.d/ext-xdebug.ini
```

Vervang de inhoud door deze inhoud:

```
[xdebug]
zend_extension="/usr/local/opt/php72-xdebug/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

Als je met PhpStorm werkt is het ook handig om de volgende regel nog toe te voegen:

```
xdebug.file_link_format="phpstorm://open?file=%f&line=%l"
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

# Startdevelopment, Stopdevelopment en Restartdevelopment

Persoonlijk wil ik niet dat bij elke reboot Apache en MySQL automatisch worden gestart.
Ik gebruik voor het starten, stoppen en herstarten van Apache en MySQL drie eenvoudige scripts:

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
/usr/local/bin/mysql.server start

# Check running processes
ps -aef | grep httpd
ps -aef | grep mysql
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
/usr/local/bin/mysql.server stop

# Start apache
sudo apachectl stop

# Check running processes
ps -aef | grep httpd
ps -aef | grep mysql
```

Sla dit bestand op.

```
$ touch /usr/local/bin/restartdevelopment
$ open -e /usr/local/bin/restartdevelopment
```

Plaats daarin de volgende code:

```
#!/bin/bash

# Restart apache
sudo apachectl -k restart

# Restart mysql
/usr/local/bin/mysql.server restart

# Check running processes
ps -aef | grep httpd
ps -aef | grep mysql
```

Sla dit bestand op.

Rechten aanpassen:

```
$ chmod +x /usr/local/bin/startdevelopment /usr/local/bin/stopdevelopment /usr/local/bin/restartdevelopment
```
