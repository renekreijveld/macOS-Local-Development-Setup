# Disclaimer

This setup works fine on my macOS machines. I am certainly no Apache, PHP and MySQL expert so should something go wrong in your setup, check the sources I used.

# Requirements

- macOS Catalina 10.15 or higher
- <a href="https://developer.apple.com/xcode/" target="_blank">XCode</a> installed

# Used sources

<a href="https://getgrav.org/blog/macos-bigsur-apache-multiple-php-versions" target="_blank">macOS 11.0 Big Sur Apache Setup: Multiple PHP Versions</a>

# First install XCode Command Line Tools

```
$ xcode-select --install
```

# Homebrew installation

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Check Homebrew installation

```
$ brew --version
Homebrew 3.0.4
Homebrew/homebrew-core (git revision fa4991; last commit 2021-03-06)
Homebrew/homebrew-cask (git revision 993c14; last commit 2021-03-06)
```

### Install wget

```
$ brew install wget
```

### Catalina/Big Sur Required Libraries

```
$ brew install openssl
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
$ brew services start httpd
```

You can also start, stop or restart Apache manually

```
$ brew services start httpd
$ brew services stop httpd
$ brew services restart httpd
```

Test Apache by going in your browser to: http://localhost:8080

## Visual Studio Code

To edit configuration files we setup a very good editor for the job: Visual Sudio Code.
Go to the <a href="https://code.visualstudio.com/" target="_blank">Visual Studio Code site</a> and click Download Mac Universal.

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
<Directory "/Users/your_user/Development/Sites"> (so WITH quotes)
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
$ brew services restart httpd
```

In your browser go to http://localhost, there the My User Web Root should appear.

# PHP Installation

First, add a tap from @shivammahtur that has many PHP versions pre-built.

```
$ brew tap shivammathur/php
```

Proceed by installing PHP versions:

```
$ brew install shivammathur/php/php@5.6
$ brew install shivammathur/php/php@7.2
$ brew install shivammathur/php/php@7.3
$ brew install shivammathur/php/php@7.4
$ brew install shivammathur/php/php@8.0
```

# Modify PHP.ini

To have webapplications work well we need to modify a number of php.ini settings.
The following values need to be modified. Search the setting in php.ini, copy the line and add a ; at the beginning of the line. Then enter the new value.

For display_errors you might want make an exception and leave that to 'On', but that is however you prefer.

```
output_buffering = Off
max_execution_time = 180
max_input_time = 180
memory_limit = 512M
display_errors = Off
post_max_size = 64M
upload_max_filesize = 64M
date.timezone = Europe/Amsterdam
```

Modify php.ini PHP 5.6:

```
$ code /usr/local/etc/php/5.6/php.ini
```

If your system doesn't have a php.ini for PHP 5.6, you can grab a copy here: <a href="https://gist.github.com/renekreijveld/a01e42bc288be56ee81cec5411c72575" target="_blank">php.ini for local development PHP 5.6</a>.

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

Modify php.ini PHP 8.0:

```
$ code /usr/local/etc/php/8.0/php.ini
```

Restart Apache after the php.ini modifications:

```
$ brew services restart httpd
```

Switch back yo the first PHP version

```
$ brew unlink php && brew link --overwrite --force php@5.6
```

At this point, you must close ALL your terminal tabs and windows. This will mean opening a new terminal to continue with the next step. This is strongly recommended because some really strange path issues can arise with existing terminals.

Quick test that we're in the correct version:

```
$ php -v
PHP 5.6.40 (cli) (built: Feb 28 2021 14:42:44)
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
#LoadModule php7_module /usr/local/opt/php@7.2/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.3/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.4/lib/httpd/modules/libphp7.so
#LoadModule php_module /usr/local/opt/php@8.0/lib/httpd/modules/libphp.so
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

I've added kick.php which is nice if you work with Akeeba Backup Kickstaert.
Save the file and restart Apache:

```
$ brew services restart httpd
```

Create a file info.php in your ~/Development/Sites folder:

```
$ echo "<?php phpinfo();" > ~/Development/Sites/info.php
```

Check if it is working by going in your browser to http://localhost/info.php

# Install PHP Switcher script

To easily switch between PHP versions we install a PHP switcher script.

```
$ curl -L https://gist.githubusercontent.com/rhukster/f4c04f1bf59e0b74e335ee5d186a98e2/raw/0c36a5067fbd63e6a36700a6aaa119df0836bdfc/sphp.sh > /usr/local/bin/sphp
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
$ sphp 7.4
Switching to php@7.4
Switching your shell
Unlinking /usr/local/Cellar/php@5.6/5.6.40... 25 symlinks removed.
Unlinking /usr/local/Cellar/php@7.2/7.2.34... 0 symlinks removed.
Unlinking /usr/local/Cellar/php@7.3/7.3.27... 0 symlinks removed.
Unlinking /usr/local/Cellar/php@7.4/7.4.16... 0 symlinks removed.
Unlinking /usr/local/Cellar/php/8.0.3... 0 symlinks removed.
Linking /usr/local/Cellar/php@7.4/7.4.16... 25 symlinks created.

If you need to have this software first in your PATH instead consider running:
  echo 'export PATH="/usr/local/opt/php@7.4/bin:$PATH"' >> ~/.profile
  echo 'export PATH="/usr/local/opt/php@7.4/sbin:$PATH"' >> ~/.profile
Switching your apache conf
Restarting apache
Stopping `httpd`... (might take a while)
==> Successfully stopped `httpd` (label: homebrew.mxcl.httpd)
==> Successfully started `httpd` (label: homebrew.mxcl.httpd)

PHP 7.4.16 (cli) (built: Mar  5 2021 00:37:36) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.16, Copyright (c), by Zend Technologies

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

# MariaDB (mysql) installation

Do note, you cannot run multiple versions of MySQL on the same machine because brew will install the database directory in the same location. So choose wisely which version you want to install.

### Install MariaDB

```
$ brew update
$ brew install mariadb
```

### Start MariaDB

```
$ brew services start mariadb
```

### Set root password to 'root'

```
$ mysql
MariaDB [(none)]> SET PASSWORD FOR root@localhost=PASSWORD('root');
MariaDB [(none)]> exit
```

### Secure MySQL

```
$ /usr/local/bin/mysql_secure_installation
```

Answers:

```
Switch to unix_socket authentication [Y/n] n
Change the root password? [Y/n] n
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```

If you need to stop the MariaDB server, you can use this command:

```
$ brew services stop mariadb
```

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
$ brew services restart httpd
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

Answer any question by simply pressing Return to accept the default values.

## APCu Configuration

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
$ brew services restart httpd
```

## APCu for other PHP versions

### PHP 7.2

```
$ sphp 7.2
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
$ brew services restart httpd
```

### PHP 7.3

```
$ sphp 7.3
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
$ brew services restart httpd
```

### PHP 7.4

```
$ sphp 7.4
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
$ brew services restart httpd
```

### PHP 8.0

```
$ sphp 8.0
$ pecl install apcu
```

Answer any question by simply pressing Return to accept the default values.

```
$ code /usr/local/etc/php/8.0/php.ini
```

Delete the line

```
extension="apcu.so"
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

```
$ code /usr/local/etc/php/8.0/conf.d/ext-apcu.ini
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
$ brew services restart httpd
```

# XDebug installation:

```
$ sphp 5.6
$ pecl install xdebug-2.5.5
```

Create a new config file for XDebug:

```
$ code /usr/local/etc/php/5.6/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension = "xdebug.so"
xdebug.remote_enable = 1
xdebug.remote_autostart = 1
xdebug.remote_host = localhost
xdebug.remote_handler = dbgp
xdebug.remote_port = 9000
```

Restart apache:

```
$ brew services restart httpd
```

In your browser go to http://localhost/info.php to ensure that XDebug is installed.

## Xdebug for other PHP versions

### PHP 7.2

```
$ sphp 7.2
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
zend_extension = "xdebug.so"
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.client_port = 9000
```

Restart apache:

```
$ brew services restart httpd
```

### PHP 7.3

```
$ sphp 7.3
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
zend_extension = "xdebug.so"
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.client_port = 9000
```

Restart apache:

```
$ brew services restart httpd
```

### PHP 7.4

```
$ sphp 7.4
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
zend_extension = "xdebug.so"
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.client_port = 9000
```

If you work with PhpStorm it is also a good idea to add the following line:

```
xdebug.file_link_format="phpstorm://open?file=%f&line=%l"
```

Restart apache:

```
$ brew services restart httpd
```

### PHP 8.0

```
$ sphp 8.0
$ pecl install xdebug
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
$ code /usr/local/etc/php/8.0/php.ini
```

Create a new config file for XDebug:

```
$ code /usr/local/etc/php/8.0/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension = "xdebug.so"
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.client_port = 9000
```

Restart apache:

```
$ brew services restart httpd
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

Personally I don't want Apache and MariaDB te start at every (re)boot automatically.
For starting, stopping and restarting Apache and MySQL I use these three simple scripts:

## startdev - Start development

```
$ code /usr/local/bin/startdev
```

Add the following code:

```
#!/bin/bash

# Start Dnsmasq
sudo brew services start dnsmasq

# Start Apache
brew services start httpd

# Start MariaDB
brew services start mariadb

# Start Mailhog
brew services start mailhog
```

## stopdev - Stop development

```
$ code /usr/local/bin/stopdev
```

Add the following code:

```
#!/bin/bash

# Stop Mailhog
brew services stop mailhog

# Stop MariaDB
brew services stop mariadb

# Stop Apache
brew services stop httpd

# Stop Dnsmasq
brew services stop dnsmasq
```

## restartdev - Restart development

```
$ code /usr/local/bin/restartdev
```

Add the following code:

```
#!/bin/bash

# Restart Mailhog
brew services restart mailhog

# Restart MariaDB
brew services restart mariadb

# Restart Apache
brew services restart httpd

# Restart Dnsmasq
brew services restart dnsmasq

```

## Modify file rights:

```
$ chmod +x /usr/local/bin/startdev /usr/local/bin/stopdev /usr/local/bin/restartdev
```
