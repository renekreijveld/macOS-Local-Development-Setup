# Disclaimer

This setup works fine on my macOS machines. I am certainly no Apache, PHP and MySQL expert so should something go wrong in your setup, check the sources I used.

# Requirements

- macOS Mojave 10.14 or higher
- <a href="https://developer.apple.com/xcode/" target="_blank">XCode</a> installed

# Used sources

<a href="https://getgrav.org/blog/macos-catalina-apache-multiple-php-versions" target="_blank">macOS 10.15 Catalina Apache Setup: Multiple PHP Versions</a>

# Install XCode Command Line Tools

```
$ xcode-select --install
```

# Homebrew installation

```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Check Homebrew installation

```
$ brew --version
Homebrew 2.0.0
Homebrew/homebrew-core (git revision 46e3; last commit 2019-01-04)
```

### Install wget

```
$ brew install wget
```

### Mojave Required Libraries

```
$ brew install openldap libiconv
```

### Update openssl
```$ brew uninstall openssl; brew uninstall openssl; brew install https://github.com/tebelorg/Tump/releases/download/v1.0.0/openssl.rb
```

## Apache installation

### Stop existing Apache and install Homebrew version

```
$ sudo apachectl stop
$ sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
$ brew install httpd
```

You can setup Apache so it starts at every (re)boot of your machine.

```
$ sudo brew services start httpd
```

You can also start, stop or restart Apache manually

```
$ sudo apachectl start
$ sudo apachectl stop
$ sudo apachectl -k restart
```

Test Apache by going in your browser to: http://localhost:8080

## Visual Studio Code

To edit configuration files we setup a very good editor for the job: Visual Sudio Code.
Go to the <a href="https://code.visualstudio.com/" target="_blank">Visual Studio Code site</a> and click Download for Mac.

Once downloaded, drag the application to your preffered Applications location. Next, you want to install the command line tools, so follow the <a href="https://code.visualstudio.com/docs/setup/mac#_launching-from-the-command-line" target="_blank">official step-by-step instructions</a> so that you can use the code command from the Terminal.

## Create a folder for your website projects

In my setup I put my website projects in my home folder submap Development/Sites so we need to create a folder for that.

```
$ mkdir -p ~/Development/Sites
```

(input your user name at 'your_user')

### Modify Apache configuration

```
$ code /usr/local/etc/httpd/httpd.conf
```

Replace:

```
Listen 8080
```

With:

```
Listen 80
```

Replace:

```
DocumentRoot "/usr/local/var/www"
```

With:

```
DocumentRoot /Users/your_user/Development/Sites (so WITHOUT quotes)
```

(input your user name at 'your_user')

Replace:

```
<Directory "/usr/local/var/www">
```

With:

```
<Directory /Users/your_user/Development/Sites> (so WITHOUT quotes)
```

(input your user name at 'your_user')

In the same <Directory ...> block:

Replace:

```
AllowOverride None
```

With:

```
AllowOverride All
```

Search for:

```
#LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so
```

And replace with:

```
LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so (so remove #)
```

### Modify User & Group

Replace:

```
User _www
Group _www
```

With:

```
User your_user
Group staff
```

(input your user name at 'your_user')

### Setup servername

Replace:

```
#ServerName www.example.com:8080
```

With:

```
ServerName localhost
```

Save the file /usr/local/etc/httpd/httpd.conf

### Create a standard index.html

```
$ echo "<h1>My User Web Root</h1>" > ~/Development/Sites/index.html
```

### Restart Apache

```
$ sudo apachectl -k restart
```

In your browser go to http://localhost, there the My User Web Root should appear.

# PHP Installation

First, add a tap for older PHP versions:

```
$ brew tap exolnet/homebrew-deprecated
```

Proceed by installing PHP versions:

```
$ brew install php@5.6
$ brew install php@7.0
$ brew install php@7.1
$ brew install php@7.2
$ brew install php@7.3
$ brew install php@7.4
```

# Modify PHP.ini

To have webapplications work good we need to modify a number of php.ini settings.
The following values need to be modified. Search the setting in php.ini, copy the line and add a ; at the beginning of the line. Then enter the new value.

For display_errors you might want make an exception and leave that to 'On', but that is however you prefer.

```
output_buffering = Off
max_execution_time = 180
max_input_time = 180
memory_limit = 512M
display_errors = Off
post_max_size = 50M
upload_max_filesize = 50M
date.timezone = Europe/Amsterdam
```

Modify php.ini PHP 5.6:

```
$ code /usr/local/etc/php/5.6/php.ini
```

If your system doesn't have a php.ini for PHP 5.6, you can grab a copy here: <a href="https://gist.github.com/renekreijveld/a01e42bc288be56ee81cec5411c72575" target="_blank">php.ini for local development PHP 5.6</a>.

Modify php.ini PHP 7.0:

```
$ code /usr/local/etc/php/7.0/php.ini
```

If your system doesn't have a php.ini for PHP 7.0, you can grab a copy here: <a href="https://gist.github.com/renekreijveld/bc3e97760d87b2dcdaa624f1587c24d6" target="_blank">php.ini for local development PHP 7.0</a>.

Modify php.ini PHP 7.1:

```
$ code /usr/local/etc/php/7.1/php.ini
```

Modify php.ini PHP 7.2:

```
$ code /usr/local/etc/php/7.2/php.ini
```

Modify php.ini PHP 7.3:

```
$ code /usr/local/etc/php/7.3/php.ini
```

Modify php.ini PHP 7.4:

```
$ code /usr/local/etc/php/7.4/php.ini
```

Restart Apache after the php.ini modifications:

```
$ sudo apachectl -k restart
```

Switch back yo the first PHP version

```
$ brew unlink php@7.4 && brew link --force --overwrite php@5.6
```

At this point, I strongly recommend closing ALL your terminal tabs and windows. This will mean opening a new terminal to continue with the next step. This is strongly recommended because some really strange path issues can arise with existing terminals.

Quick test that we're in the correct version:

```
$ php -v
PHP 5.6.39 (cli) (built: Dec  7 2018 08:27:47) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
```

# Apache PHP Setup

### Modify Apache configuration

```
$ code /usr/local/etc/httpd/httpd.conf
```

Find the line that loads the mod_rewrite module:

LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so

Below this add the following libphp modules:

```
LoadModule php5_module /usr/local/opt/php@5.6/lib/httpd/modules/libphp5.so
#LoadModule php7_module /usr/local/opt/php@7.0/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.1/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.2/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.3/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.4/lib/httpd/modules/libphp7.so
```

Replace:

```
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

With:

```
<IfModule dir_module>
    DirectoryIndex kick.php index.php index.html
</IfModule>

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```

Save the file and restart Apache:

```
$ sudo apachectl -k stop
$ sudo apachectl start
```

Create a file info.php in your ~/Development/Sites folder:

```
$ echo "<?php phpinfo();" > ~/Development/Sites/info.php
```

Check if it is working by going in your browser to http://localhost/info.php

# Install PHP Switcher script

To easily switch between PHP versions we install a PHP switcher script.

```
$ curl -L https://gist.githubusercontent.com/rhukster/f4c04f1bf59e0b74e335ee5d186a98e2/raw > /usr/local/bin/sphp
$ chmod +x /usr/local/bin/sphp
```

# Check your local path

```
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

If you don't see this, you might need to add these manually to your path. Depending on your shell your using, you may need to add this line to ~/.profile, ~/.bash_profile, or ~/.zshrc. We will assume you are using the default bash shell, so add this line to a your .profile (create it if it doesn't exist) file at the root of your user directory:

```
$ code ~/.profile
export PATH=/usr/local/bin:/usr/local/sbin:$PATH
```

Then stop and restart the Terminal application.

### Testing the PHP Switching

Test the switcher script:

```
$ sphp 7.1
```

```
$ sphp 7.1
Switching to php@7.1
Switching your shell
Unlinking /usr/local/Cellar/php@5.6/5.6.39... 25 symlinks removed
Unlinking /usr/local/Cellar/php@7.1/7.1.25... 0 symlinks removed
Unlinking /usr/local/Cellar/php@7.2/7.2.13... 0 symlinks removed
Unlinking /usr/local/Cellar/php/7.3.0... 0 symlinks removed
Unlinking /usr/local/Cellar/php/7.4.1... 0 symlinks removed
Linking /usr/local/Cellar/php@7.1/7.1.25... 25 symlinks created

If you need to have this software first in your PATH instead consider running:
  echo 'export PATH="/usr/local/opt/php@7.1/bin:$PATH"' >> ~/.bash_profile
  echo 'export PATH="/usr/local/opt/php@7.1/sbin:$PATH"' >> ~/.bash_profile
You will need sudo power from now on
Switching your apache conf
Restarting apache

PHP 7.1.25 (cli) (built: Dec  7 2018 08:20:45) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.1.25, Copyright (c) 1999-2018, by Zend Technologies
    with Xdebug v2.6.1, Copyright (c) 2002-2018, by Derick Rethans

All done!
```

Refresh the page <a href="http://localhost/info.php" target="_blank">http://localhost/info.php</a> in your browser.

# Updating software

Brew makes it super easy to update PHP and the other packages you install. The first step is to update Brew so that it gets a list of available updates:

```
$ brew update
```

This will spit out a list of available updates, and any deleted formulas. To upgrade the packages simply type:

```
$ brew upgrade
```

To update all of your PHP versions you have to switch to them with the sphp script, and then run brew update.

# MySQL installation

Do note, you cannot run multiple versions of MySQL on the same machine because brew will install the database directory 
in the same location. So choose wisely which version you want to install.

### Install MariaDB

```
$ brew update
$ brew install mariadb
```

### Start MariaDB

```
$ brew services start mariadb
```

### Secure MySQL

```
$ /usr/local/bin/mysql_secure_installation
```

Answers:

```
Would you like to setup VALIDATE PASSWORD plugin? Press y|Y for Yes, any other key for No: Enter (= no)
Please set the password for root here. New password: root
Re-enter new password: root
Remove anonymous users? (Press y|Y for Yes, any other key for No) : Y
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : Y
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : Y
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y
```

If you need to stop the MariaDB server, you can use this command:

```
$ brew services stop mariadb
```

# Sequel Pro Installation

For easy management of the MySQL databases we can user the excellent free tool Sequel Pro.
Download and install it from <a href="https://www.sequelpro.com/" target="_blank">https://www.sequelpro.com/</a>.

After installation start Sequel Pro. You can connect to MYSQL by inputting the following settings:

```
Host: 127.0.0.1
Username: root
Password: root
Database: <leave empty>
Port: <leave empty>
```

After logging in you can create an empty database by click on 'Choose Database...' at the top left and then choose 'Add Database...'.

# Apache Virtual Hosts

### Modify Apache configuration

```
$ code /usr/local/etc/httpd/httpd.conf
```

Replace:

```
#LoadModule vhost_alias_module lib/httpd/modules/mod_vhost_alias.so
```

With:

```
LoadModule vhost_alias_module lib/httpd/modules/mod_vhost_alias.so
```

Replace:

```
#Include /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

With:

```
Include /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

Modify httpd-vhosts.conf:

```
$ code /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

Remove all existing lines below the comments block and add the following lines:

```
<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Development/Sites"
    ServerName localhost
    ErrorLog "/usr/local/var/log/httpd/error_log"
    CustomLog "/usr/local/var/log/httpd/access_log" common
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

(input your user name at 'your_user')

# Dnsmasq installation

We can now very easily add a new virtual host.
By creating a subfolder in ~/Development/Sites/, for example, 'testsite', this new website is immediately accessible through the domain name 'testsite.test'.

But we need to modify DNS so it resolves to this domain. Therefor we install Dnsmasq.

```
$ brew install dnsmasq
```

Setup *.test hosts:

```
$ echo 'address=/.test/127.0.0.1' >> /usr/local/etc/dnsmasq.conf
```

Start Dnsmasq and make sure it starts at every reboot:

```
$ sudo brew services start dnsmasq
```

Add to resolvers:

```
$ sudo mkdir -v /etc/resolver
$ sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/test'
```

Test this by pinging to an unknown .test name. There should come a reply from 127.0.0.1:

```
$ ping newwebsite.test
PING newwebsite.test (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.039 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.160 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.093 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.144 ms
```

Restart apache:

```
$ sudo apachectl -k restart
```

# APC Cache installation:

To have PHP run faster we install APCu Cache. Zend OPcache was already installed with the PHP installation.

```
$ sphp 5.6
$ brew install autoconf
```

When we have installed autoconf we can install APCu via PECL. PECL is a PHP package manager that is now the preferred way to install PHP packages.

```
$ pecl channel-update pecl.php.net
$ pecl install apcu-4.0.11
```

## APCu Configuration

```
$ code /usr/local/etc/php/5.6/php.ini
```

Delete the line

```
extension="apcu.so" 
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

```
$ code /usr/local/etc/php/5.6/conf.d/ext-apcu.ini
```

Put the following contents in that file:

```
[apcu]
extension="apcu.so"
apc.enabled=1
apc.shm_size=64M
apc.ttl=7200
apc.enable_cli=1
```

Save and close the file and restart Apache:

```
$ sudo apachectl -k restart
```

## APCu for other PHP versions

### PHP 7.0

```
$ sphp 7.0
$ pecl uninstall -r apcu
$ pecl install apcu
```

Answer any question by simply pressing Return to accept the default values.

```
$ code /usr/local/etc/php/7.0/php.ini
```

Delete the line

```
extension="apcu.so" 
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

```
$ code /usr/local/etc/php/7.0/conf.d/ext-apcu.ini
```

Put the following contents in that file:

```
[apcu]
extension="apcu.so"
apc.enabled=1
apc.shm_size=64M
apc.ttl=7200
apc.enable_cli=1
```

Save and close the file and restart Apache:

```
$ sudo apachectl -k restart
```

### PHP 7.1

```
$ sphp 7.1
$ pecl uninstall -r apcu
$ pecl install apcu
```

Answer any question by simply pressing Return to accept the default values.

```
$ code /usr/local/etc/php/7.1/php.ini
```

Delete the line

```
extension="apcu.so" 
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

```
$ code /usr/local/etc/php/7.1/conf.d/ext-apcu.ini
```

Put the following contents in that file:

```
[apcu]
extension="apcu.so"
apc.enabled=1
apc.shm_size=64M
apc.ttl=7200
apc.enable_cli=1
```

Save and close the file and restart Apache:

```
$ sudo apachectl -k restart
```

### PHP 7.2

```
$ sphp 7.2
$ pecl uninstall -r apcu
$ pecl install apcu
```

Answer any question by simply pressing Return to accept the default values.

```
$ code /usr/local/etc/php/7.2/php.ini
```

Delete the line

```
extension="apcu.so" 
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

```
$ code /usr/local/etc/php/7.2/conf.d/ext-apcu.ini
```

Put the following contents in that file:

```
[apcu]
extension="apcu.so"
apc.enabled=1
apc.shm_size=64M
apc.ttl=7200
apc.enable_cli=1
```

Save and close the file and restart Apache:

```
$ sudo apachectl -k restart
```

### PHP 7.3

```
$ sphp 7.3
$ pecl uninstall -r apcu
$ pecl install apcu
```

Answer any question by simply pressing Return to accept the default values.

```
$ code /usr/local/etc/php/7.3/php.ini
```

Delete the line

```
extension="apcu.so" 
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

```
$ code /usr/local/etc/php/7.3/conf.d/ext-apcu.ini
```

Put the following contents in that file:

```
[apcu]
extension="apcu.so"
apc.enabled=1
apc.shm_size=64M
apc.ttl=7200
apc.enable_cli=1
```

Save and close the file and restart Apache:

```
$ sudo apachectl -k restart
```

### PHP 7.4

```
$ sphp 7.4
$ pecl uninstall -r apcu
$ pecl install apcu
```

Answer any question by simply pressing Return to accept the default values.

```
$ code /usr/local/etc/php/7.4/php.ini
```

Delete the line

```
extension="apcu.so" 
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

```
$ code /usr/local/etc/php/7.4/conf.d/ext-apcu.ini
```

Put the following contents in that file:

```
[apcu]
extension="apcu.so"
apc.enabled=1
apc.shm_size=64M
apc.ttl=7200
apc.enable_cli=1
```

Save and close the file and restart Apache:

```
$ sudo apachectl -k restart
```

# XDebug installation:

```
$ sphp 5.6
$ pecl install xdebug-2.5.5
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
$ code /usr/local/etc/php/5.6/php.ini
```

Create a new config file for XDebug:

```
$ code /usr/local/etc/php/5.6/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension="xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

Restart apache:

```
$ sudo apachectl -k restart
```

In your browser go to http://localhost/info.php to ensure that XDebug is installed.

## Xdebug for other PHP versions

### PHP 7.0

```
$ sphp 7.0
$ pecl uninstall -r xdebug
$ pecl install xdebug
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
$ code /usr/local/etc/php/7.0/php.ini
```

Create a new config file for XDebug:

```
$ code /usr/local/etc/php/7.0/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension="xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

Restart apache:

```
$ sudo apachectl -k restart
```

### PHP 7.1

```
$ sphp 7.1
$ pecl uninstall -r xdebug
$ pecl install xdebug
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
$ code /usr/local/etc/php/7.1/php.ini
```

Create a new config file for XDebug:

```
$ code /usr/local/etc/php/7.1/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension="xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

Restart apache:

```
$ sudo apachectl -k restart
```

### PHP 7.2

```
$ sphp 7.2
$ pecl uninstall -r xdebug
$ pecl install xdebug
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
$ code /usr/local/etc/php/7.2/php.ini
```

Create a new config file for XDebug:

```
$ code /usr/local/etc/php/7.2/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension="xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

If you work with PhpStorm it is also a good idea to add the following line:

```
xdebug.file_link_format="phpstorm://open?file=%f&line=%l"
```

Restart apache:

```
$ sudo apachectl -k restart
```

### PHP 7.3

```
$ sphp 7.3
$ pecl uninstall -r xdebug
$ pecl install xdebug
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
$ code /usr/local/etc/php/7.3/php.ini
```

Create a new config file for XDebug:

```
$ code /usr/local/etc/php/7.3/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension="xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

Restart apache:

```
$ sudo apachectl -k restart
```

### PHP 7.4

```
$ sphp 7.4
$ pecl uninstall -r xdebug
$ pecl install xdebug
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
$ code /usr/local/etc/php/7.4/php.ini
```

Create a new config file for XDebug:

```
$ code /usr/local/etc/php/7.4/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension="xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

Restart apache:

```
$ sudo apachectl -k restart
```

# Mailhog

MailHog is a small application which intercepts email sent out of your sites and keeps it locally. You can use a web interface to review the mail. This comes in handy when testing the email features of the sites you are building without risking any email accidentally escaping to the wild.

```
$ brew install mailhog
$ brew services start mailhog
```

Mailhog runs a SMTP mailserver at port 1025. To intercept all outgoing emails from a local Joomla website for example setup tour Joomla mail settings as follows:

```
Send Mail: yes
From Email: <sending emailadrress>
From Name: <sender name>
Mailer: SMTP
SMTP Host: 127.0.0.1
SMTP Port: 1025
SMTP Security: None
SMTP Authentication: No
```

Open your webbrowser at <a href="http://127.0.0.1:8025" target="_blank">http://127.0.0.1:8025</a> and see all outgoing emails collected there. Emails are not sent to the Internet.

# Startdevelopment, Stopdevelopment and Restartdevelopment

Personally I don't want Apache and MySQL te start at every (re)boot automatically.
For starting, stopping and restarting Apache and MySQL I use these three simple scripts:

## Startdevelopment

```
$ code /usr/local/bin/startdevelopment
```

Add the following code:

```
#!/bin/bash

# start DNSMasq
sudo brew services start dnsmasq
echo "DNSMasq started"

# start mariadb
brew services start mariadb
echo "Mariadb started"

# start apache
sudo apachectl start
echo "Apache started"

# start mailhog
brew services start mailhog
echo "Mailhog started"
```

## Stopdevelopment

```
$ code /usr/local/bin/stopdevelopment
```

Add the following code:

```
#!/bin/bash

# stop mailhog
brew services stop mailhog
echo "Mailhog stopped"

# stop apache
sudo apachectl stop
echo "Apache stopped"

# stop mariadb
brew services stop mariadb
echo "Mariadb stopped"

# stop DNSMasq
sudo brew services stop dnsmasq
echo "DNSMasq stopped"
```

## Restartdevelopment

```
$ code /usr/local/bin/restartdevelopment
```

Add the following code:

```
#!/bin/bash

# restart mailhog
brew services restart mailhog
echo "Mailhog restarted"

# stop apache
sudo apachectl -k restart
echo "Apache restarted"

# stop mariadb
brew services restart mariadb
echo "Mariadb restarted"

# stop DNSMasq
sudo brew services restart dnsmasq
echo "DNSMasq restarted"
```

## Modify file rights:

```
$ chmod +x /usr/local/bin/startdevelopment /usr/local/bin/stopdevelopment /usr/local/bin/restartdevelopment
```
