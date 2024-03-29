## Disclaimer

This setup works fine on my macOS machines. I am certainly no Apache, PHP and MariaDB expert so should something go wrong in your setup, check the sources I used.

## Requirements

- macOS Ventura

## Used sources

<a href="https://getgrav.org/blog/macos-ventura-apache-multiple-php-versions" target="_blank">macOS 13.0 Ventura Apache Setup: Multiple PHP Versions</a>

## Install XCode Command Line Tools

```
xcode-select --install
```

## Homebrew installation

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Check Homebrew installation

```
brew --version
Homebrew 4.0.1
Homebrew/homebrew-core N/A
```

## Install wget and zip

```
brew install wget zip
```

## Apache installation

### Stop existing Apache and install Homebrew version

```
sudo apachectl stop
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
brew install httpd
```

You can setup Apache so it starts at every (re)boot of your machine.
==> Fetching wget

```
brew services start httpd
```

You can also start, stop or restart Apache manually

```
brew services start httpd
brew services stop httpd
brew services restart httpd
```

Test Apache by going in your browser to: http://localhost:8080

## Visual Studio Code

To edit configuration files we setup Visual Sudio Code.
Go to the <a href="https://code.visualstudio.com/" target="_blank">Visual Studio Code site</a> and click Download Mac Universal.

Once downloaded, drag the application to your preffered Applications location. Next, you want to install the command line tools, so follow the <a href="https://code.visualstudio.com/docs/setup/mac#_launching-from-the-command-line" target="_blank">official step-by-step instructions</a> so that you can use the code command from the Terminal.

## Create a folder for your website projects

In my setup I put my website projects in my home folder submap Development/Sites so we need to create a folder for that.

```
$ mkdir -p ~/Development/Sites
```

### Modify Apache configuration

```
code /usr/local/etc/httpd/httpd.conf
```

Replace:

```
Listen 8080
```

with:

```
Listen 80
```

Replace:

```
DocumentRoot "/usr/local/var/www"
```

with:

```
DocumentRoot "/Users/your_user/Development/Sites"
```

Replace your_user with your own macOS username.

Replace:

```
<Directory "/usr/local/var/www">
```

with:

```
<Directory "/Users/your_user/Development/Sites"> (so WITH quotes)
```

Replace your_user with your own macOS username.

In the same <Directory ...> block:

Replace:

```
AllowOverride None
```

with:

```
AllowOverride All
```

Search for:

```
#LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so
```

and replace with:

```
LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so (so remove #)
```

### Modify User & Group

Replace:

```
User _www
Group _www
```

with:

```
User your_user
Group staff
```

Replace your_user with your own macOS username.

### Setup servername

Replace:

```
#ServerName www.example.com:8080
```

with:

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
brew services restart httpd
```

In your browser go to http://localhost, there the My User Web Root should appear.

## PHP Installation

First, add a tap from @shivammahtur that has many PHP versions pre-built.

```
brew tap shivammathur/php
```

Proceed by installing PHP versions:

```
brew install shivammathur/php/php@5.6
brew install shivammathur/php/php@7.4
brew install shivammathur/php/php@8.0
brew install shivammathur/php/php@8.1
brew install shivammathur/php/php@8.2
```

## Modify PHP.ini

To have webapplications work well we need to modify a number of php.ini settings.
The following values need to be modified for every installed PHP version. Search the setting in php.ini and enter the new value.

For display_errors you might want make an exception and leave that to 'On', but that is however you prefer.

```
output_buffering = Off
max_execution_time = 180
max_input_time = 180
memory_limit = 512M
display_errors = Off
post_max_size = 256M
upload_max_filesize = 256M
max_file_uploads = 50
date.timezone = Europe/Amsterdam
```

Modify php.ini PHP 5.6:

```
code /usr/local/etc/php/5.6/php.ini
```

If your system doesn't have a php.ini for PHP 5.6, you can grab a copy here: <a href="https://gist.github.com/renekreijveld/a01e42bc288be56ee81cec5411c72575" target="_blank">php.ini for local development PHP 5.6</a>.


Modify php.ini PHP 7.4:

```
code /usr/local/etc/php/7.4/php.ini
```

Modify php.ini PHP 8.0:

```
code /usr/local/etc/php/8.0/php.ini
```

Modify php.ini PHP 8.1:

```
code /usr/local/etc/php/8.1/php.ini
```

Modify php.ini PHP 8.2:

```
code /usr/local/etc/php/8.2/php.ini
```
Restart Apache after the php.ini modifications:

```
brew services restart httpd
```

At this point, you must close ALL your terminal tabs and windows. This will mean opening a new terminal to continue with the next step. This is strongly recommended because some really strange path issues can arise with existing terminals.

Switch back yo the first PHP version

```
brew unlink php && brew link --overwrite --force php@5.6
```

Quick test that we're in the correct version:

```
php -v
PHP 5.6.40 (cli) (built: Feb 28 2021 14:42:44)
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
```

## PHP setup in the Apache configuration

```
code /usr/local/etc/httpd/httpd.conf
```

Find the line that loads the mod_rewrite module:

```
LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so
```

Below this add the following libphp modules:

```
LoadModule php5_module /usr/local/opt/php@5.6/lib/httpd/modules/libphp5.so
#LoadModule php7_module /usr/local/opt/php@7.4/lib/httpd/modules/libphp7.so
#LoadModule php_module /usr/local/opt/php@8.0/lib/httpd/modules/libphp.so
#LoadModule php_module /usr/local/opt/php@8.1/lib/httpd/modules/libphp.so
#LoadModule php_module /usr/local/opt/php@8.2/lib/httpd/modules/libphp.so
```

Replace:

```
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

with:

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
brew services restart httpd
```

Create a file info.php in your ~/Development/Sites folder:

```
echo "<?php phpinfo();" > ~/Development/Sites/info.php
```

Check if it is working by going in your browser to http://localhost/info.php

## Install PHP Switcher script

To easily switch between PHP versions we install a PHP switcher script.

```
curl -L https://gist.githubusercontent.com/rhukster/f4c04f1bf59e0b74e335ee5d186a98e2/raw/7227b6e8eab67fbdb5bd1053d7d761eb1507e8ac/sphp.sh > /usr/local/bin/sphp
chmod +x /usr/local/bin/sphp
```

## Testing the PHP Switching

Test the switcher script:

```
sphp 7.4
Switching to php@7.4
Switching your shell
Unlinking /usr/local/Cellar/php@5.6/5.6.40_6... 0 symlinks removed.
Unlinking /usr/local/Cellar/php@7.4/7.4.33_1... 0 symlinks removed.
Unlinking /usr/local/Cellar/php@8.0/8.0.28... 25 symlinks removed.
Unlinking /usr/local/Cellar/php@8.1/8.1.16... 0 symlinks removed.
Unlinking /usr/local/Cellar/php/8.2.3... 0 symlinks removed.
Linking /usr/local/Cellar/php@7.4/7.4.33_1... 25 symlinks created.

If you need to have this software first in your PATH instead consider running:
  echo 'export PATH="/usr/local/opt/php@7.4/bin:$PATH"' >> /Users/renekreijveld/.bash_profile
  echo 'export PATH="/usr/local/opt/php@7.4/sbin:$PATH"' >> /Users/renekreijveld/.bash_profile
Switching your apache conf
Restarting apache
Stopping `httpd`... (might take a while)
==> Successfully stopped `httpd` (label: homebrew.mxcl.httpd)
==> Successfully started `httpd` (label: homebrew.mxcl.httpd)

PHP 7.4.33 (cli) (built: Feb 15 2023 06:33:21) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.33, Copyright (c), by Zend Technologies

All done!
```

Refresh the page <a href="http://localhost/info.php" target="_blank">http://localhost/info.php</a> in your browser.

## Updating Brew software

Brew makes it super easy to update PHP and the other packages you install. The first step is to update Brew so that it gets a list of available updates:

```
brew update
```

This will spit out a list of available updates, and any deleted formulas. To upgrade the packages simply type:

```
brew upgrade
```

To update all of your PHP versions you have to switch to them with the sphp script, and then run brew update.

## MariaDB (mysql) installation

Do note, you cannot run multiple versions of MySQL on the same machine because brew will install the database directory in the same location. So choose wisely which version you want to install.

### Install MariaDB

```
brew update
brew install mariadb
```

### Start MariaDB

```
brew services start mariadb
```

### Set root password to 'root'

```
mysql
MariaDB [(none)]> SET PASSWORD FOR root@localhost = PASSWORD('root');
MariaDB [(none)]> EXIT;
```

### Securing MariaDB

```
/usr/local/bin/mysql_secure_installation
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
brew services stop mariadb
```

## Adminer installation

Now that we have PHP and MariaDB running we need a tool to administrate our MariaDB database. Two native macOS applications that are quite good are <a href="https://www.sequelpro.com/" target="_blank">Sequel Pro</a> and <a href="https://dbeaver.io/" target="_blank">DBeaver</a>.

But personally I prefer Adminer, a PHP based tool that runs in the browser. You can download Adminer from the <a href="https://www.adminer.org/#download" target="_blank">Adminer.org website</a>

The most minimalistic version the version for MySQL, English only. Download the file and save it as adminer.php in your Development/Sites folder. You can then open it at <a href="http://localhost/adminer.php" target="_blank">http://localhost/adminer.php</a>.

Then login with username 'root' and password 'root' (without the quotes).

## Dnsmasq installation

We can now very easily add a new virtual host.
By creating a subfolder in ~/Development/Sites/, for example, 'testsite', this new website is immediately accessible through the domain name 'testsite.test'.

But we need to modify DNS so it resolves to this domain. Therefor we install Dnsmasq.

```
brew install dnsmasq
```

Setup *.dev.test hosts:

```
echo 'address=/.dev.test/127.0.0.1' >> /usr/local/etc/dnsmasq.conf
```

Start Dnsmasq and make sure it starts at every reboot:

```
sudo brew services start dnsmasq
```

Add to resolvers:

```
sudo mkdir -v /etc/resolver
sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/test'
```

Test this by pinging to an unknown .dev.test name. Thex§re should come a reply from 127.0.0.1:

```
ping newsite.dev.test
PING newsite.dev.test (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.102 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.274 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.214 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.255 ms
```

Restart apache:

```
brew services restart httpd
```

## APCu Cache installation:

To have PHP run faster we install APCu Cache. Zend OPcache was already installed with the PHP installation.

### PHP 5.6

```
sphp 5.6
brew install autoconf
```

When we have installed autoconf we can install APCu via PECL. PECL is a PHP package manager that is now the preferred way to install PHP packages.

```
pecl channel-update pecl.php.net
pecl install apcu-4.0.11
```

Answer any question by simply pressing Return to accept the default values.

### APCu Configuration

First, remove the first line that was added to php.ini

```
code /usr/local/etc/php/5.6/php.ini
```

Delete the line

```
extension="apcu.so"
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

```
code /usr/local/etc/php/5.6/conf.d/ext-apcu.ini
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

Save and close the file.

### PHP 7.4

```
sphp 7.4
pecl install apcu
```

Answer any question by simply pressing Return to accept the default values.

```
code /usr/local/etc/php/7.4/php.ini
```

Delete the line

```
extension="apcu.so"
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

```
code /usr/local/etc/php/7.4/conf.d/ext-apcu.ini
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

Save and close the file.

### PHP 8.0

```
sphp 8.0
pecl install apcu
```

Answer any question by simply pressing Return to accept the default values.

```
code /usr/local/etc/php/8.0/php.ini
```

Delete the line

```
extension="apcu.so"
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

```
code /usr/local/etc/php/8.0/conf.d/ext-apcu.ini
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

Save and close the file.

### PHP 8.1

```
sphp 8.1
pecl install apcu
```

Answer any question by simply pressing Return to accept the default values.

```
code /usr/local/etc/php/8.1/php.ini
```

Delete the line

```
extension="apcu.so"
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

```
code /usr/local/etc/php/8.1/conf.d/ext-apcu.ini
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
brew services restart httpd
```

### PHP 8.2

```
sphp 8.2
pecl install apcu
```

Answer any question by simply pressing Return to accept the default values.

```
code /usr/local/etc/php/8.2/php.ini
```

Delete the line

```
extension="apcu.so"
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

```
code /usr/local/etc/php/8.2/conf.d/ext-apcu.ini
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

Save and close the file.

## XDebug installation:

```
sphp 5.6
pecl install xdebug-2.5.5
```

```
code /usr/local/etc/php/5.6/php.ini
```

Delete the line

```
zend_extension="xdebug.so"
```

that was added at the top of php.ini. Save and close php.ini. Then create a new separate .ini file for APCu:

Create a new config file for XDebug:

```
code /usr/local/etc/php/5.6/conf.d/ext-xdebug.ini
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
brew services restart httpd
```

In your browser go to http://localhost/info.php to ensure that XDebug is installed.

## Xdebug for other PHP versions

### PHP 7.4

```
sphp 7.4
pecl install xdebug-3.1.6
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
code /usr/local/etc/php/7.4/php.ini
```

Create a new config file for XDebug:

```
code /usr/local/etc/php/7.4/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension = "xdebug.so"
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.client_port = 9003
xdebug.idekey = "PHPSTORM"
```

Save and close the file.

### PHP 8.0

```
sphp 8.0
pecl install xdebug
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
code /usr/local/etc/php/8.0/php.ini
```

Create a new config file for XDebug:

```
code /usr/local/etc/php/8.0/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension = "xdebug.so"
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.client_port = 9003
xdebug.idekey = "PHPSTORM"
```

Save and close the file.

### PHP 8.1

```
sphp 8.1
pecl install xdebug
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
code /usr/local/etc/php/8.1/php.ini
```

Create a new config file for XDebug:

```
code /usr/local/etc/php/8.1/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension = "xdebug.so"
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.client_port = 9003
xdebug.idekey = "PHPSTORM"
```

Save and close the file.

### PHP 8.2

```
sphp 8.2
pecl install xdebug
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
code /usr/local/etc/php/8.2/php.ini
```

Create a new config file for XDebug:

```
code /usr/local/etc/php/8.2/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension = "xdebug.so"
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.client_port = 9003
xdebug.idekey = "PHPSTORM"
```

Save and close the file and restart Apache

```
brew services restart httpd
```

## Mailhog

MailHog is a small application which intercepts email sent out of your sites and keeps it locally. You can use a web interface to review the mail. This comes in handy when testing the email features of the sites you are building without risking any email accidentally escaping to the wild.

```
brew install mailhog
brew services start mailhog
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

### Setup mhsendmail

For even easier user of Mailhog we install mhsendmail and configure that in php.ini. This way emails sent through PHP will automatically be caught by Mailhog.
Install go:

```
brew install go
```

Install mhsendmail through go:

```
go install github.com/mailhog/mhsendmail@latest
```

Move the mhsendmail binary to /usr/local/bin:

```
mv ./go/bin/mhsendmail /usr/local/bin
```

Now all we need to do is configure mhsendmail in php.ini. Edit the php.ini's:

```
code /usr/local/etc/php/5.6/php.ini
code /usr/local/etc/php/7.4/php.ini
code /usr/local/etc/php/8.0/php.ini
code /usr/local/etc/php/8.1/php.ini
code /usr/local/etc/php/8.2/php.ini
```

Once you have php.ini loaded in Visual Studio Code find the line with:

```
;sendmail_path =
```

and change that to:

```
sendmail_path = /usr/local/bin/mhsendmail
```

Save and close php.ini and restart apache:

```
brew services restart httpd
```

## Setting up SSL

To use a local SSL certificate we need to install two tools.

```
brew install mkcert nss
```

Next we have to install the server and run it (enter your password when prompted):

```
mkcert -install
```

You should get the following message:

```
The local CA is now installed in the system trust store!
```

Create a location for the certificates:

```
cd /usr/local/etc/httpd
mkdir certs && cd certs
```

Generate a certificate for localhost:

```
mkcert localhost
```

Generate a wildcart certificate for *.dev.test:

```
mkcert "*.dev.test"
```

Now we need to modify our Apache SSL configuration:

```
code /usr/local/etc/httpd/httpd.conf
```

Uncomment the line with:

```
#LoadModule socache_shmcb_module lib/httpd/modules/mod_socache_shmcb.so
```

So, remove the # at the beginning of the line.

Uncomment the line with:

```
#LoadModule ssl_module lib/httpd/modules/mod_ssl.so
```

Here too, remove the # at the beginning of the line.

Uncomment the line with:

```
#Include /usr/local/etc/httpd/extra/httpd-ssl.conf
```

Here too, remove the # at the beginning of the line. Save and close the file.

Modify the httpd-ssl configuration:

```
code /usr/local/etc/httpd/extra/httpd-ssl.conf
```

Change the line with:

```
Listen 8443
```

to

```
Listen 443
```

Then, find the line that starts with:

```
<VirtualHost _default_:8443>
```

and delete that line and all lines below it. So the file should end with:

```
##
## SSL Virtual Host Context
##
```

Save and close the file.

## Apache Virtual Hosts

### Modify Apache configuration

```
code /usr/local/etc/httpd/httpd.conf
```

Replace:

```
#LoadModule vhost_alias_module lib/httpd/modules/mod_vhost_alias.so
```

with:

```
LoadModule vhost_alias_module lib/httpd/modules/mod_vhost_alias.so
```

Replace:

```
#Include /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

with:

```
Include /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

Modify httpd-vhosts.conf:

```
code /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

Remove all existing lines below the comments block and add the following lines:

```
<Directory "/Users/your_user/Development/Sites">
    Allow From All
    AllowOverride All
    Options +Indexes
    Require all granted
</Directory>

<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Development/Sites"
    ServerName localhost
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
    ErrorLog "/usr/local/var/log/httpd/error_log"
    CustomLog "/usr/local/var/log/httpd/access_log" common
</VirtualHost>

<VirtualHost *:443>
    DocumentRoot "/Users/your_user/Development/Sites"
    ServerName localhost
    SSLEngine on
    SSLCertificateFile "/usr/local/etc/httpd/certs/localhost.pem"
    SSLCertificateKeyFile "/usr/local/etc/httpd/certs/localhost-key.pem"
</VirtualHost>

<Virtualhost *:80>
    VirtualDocumentRoot "/Users/your_user/Development/Sites/%1"
    ServerAlias *.dev.test
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
    UseCanonicalName Off
</Virtualhost>

<VirtualHost *:443>
    VirtualDocumentRoot "/Users/your_user/Development/Sites/%1"
    ServerAlias *.dev.test
    SSLEngine on
    SSLCertificateFile "/usr/local/etc/httpd/certs/_wildcard.dev.test.pem"
    SSLCertificateKeyFile "/usr/local/etc/httpd/certs/_wildcard.dev.test-key.pem"
</VirtualHost>
```

Replace your_user everywhere with your own macOS username.

## Restart Apache

```
brew services restart httpd
```

## Startdev, Stopdev and Restartdev

For easy starting, stopping and restarting the proicesses for Apache and MySQL and Mailhog I use these three simple scripts. Add these scripts to a folder that is inside your path, for example in /usr/local/bin or in /usr/local/bin :

### startdev - Start development

```
code /usr/local/bin/startdev
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

### stopdev - Stop development

```
code /usr/local/bin/stopdev
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
sudo brew services stop dnsmasq
```

### restartdev - Restart development

```
code /usr/local/bin/restartdev
```

Add the following code:

```
#!/bin/bash

# Restart Apache
brew services restart httpd

# Restart MariaDB
brew services restart mariadb

# Restart Mailhog
brew services restart mailhog

# Restart Dnsmasq
sudo brew services restart dnsmasq
```

### Modify file rights:

```
chmod +x /usr/local/bin/startdev /usr/local/bin/stopdev /usr/local/bin/restartdev
```

## Finished

Your local Apache / PHP / MariaDB / Mailhog setup is now ready for use.
