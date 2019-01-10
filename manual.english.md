# General use

After a (re)boot of your Mac, depending on the setup Apache and MySQL are already running.
In my personal setup I have to start these manually. I open a terminal and start the services:

```
$ startdevelopment
```

Apache, MariaDB, DNSMasq and Mailhog are now ready for use.

# Create a new database

- Create a new database with Sequel Pro

# Create a new website

Creating a new website is very simple. In the Sites folder create a new subfolder for your website.

As an example create a folder **testwebsite**.

```
$ mkdir ~/Development/Sites/testwebsite
```

Then add your html/php/whatever files in the folder ~/Development/Sites/testwebsite.

There is nothing else to configure. The website is immediately ready in your browser at http://testwebsite.test

# Stop Development

Apache, MariaDB, DNSMasq and Mailhog can be stopped with the terminal command:

```
$ stopdevelopment
```

# Restart development

Apache, MariaDB, DNSMasq and Mailhog can be restarted with the terminal command:

```
$ restartdevelopment
```

# Reference

## Apache

Install Apache as service so it automatically starts at every (re)boot

```
$ sudo brew services start httpd
```

Remove Apache service so it is no longer automatically startet at every (re)boot

```
$ sudo brew services stop httpd
```

Start apache:

```
$ sudo apachectl start
```

Stop apache:

```
$ sudo apachectl stop
```

Restart apache:

```
$ sudo apachectl -k restart
```

Edit httpd.conf:

```
$ code /usr/local/etc/httpd/httpd.conf
```

Check if Apache is running:

```
$ ps -aef | grep httpd
```

# PHP

Edit php.ini 5.6:

```
$ code /usr/local/etc/php/5.6/php.ini
```

Edit php.ini 7.1:

```
$ code /usr/local/etc/php/7.1/php.ini
```

Edit php.ini 7.2:

```
$ code /usr/local/etc/php/7.2/php.ini
```

Edit php.ini 7.3:

```
$ code /usr/local/etc/php/7.3/php.ini
```

Change to php 5.6:

```
$ sphp 5.6
```

Change to php 7.1:

```
$ sphp 7.1
```

Change to php 7.2:

```
$ sphp 7.2
```

Change to php 7.3:

```
$ sphp 7.3
```

# MariaDB

Start MariaDB at every (re)boot:

```
$ brew services start mariadb
```

Stop MariaDB at every (re)boot:

```
$ brew services stop mariadb
```

Check if MariaDB is running:

```
$ ps -aef | grep mysql
```
