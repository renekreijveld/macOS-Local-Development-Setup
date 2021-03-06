# General use

After a (re)boot of your Mac, depending on the setup Apache and MariaDB are already running.
In my personal setup I have to start these manually. I open a terminal and start the services:

```
$ startdev
```

Apache, MariaDB, DNSMasq and Mailhog are now ready for use.

# Create a new database

- Create a new database with Sequel Pro, DBeaver, Adminer or whatever tool you user to administrate your MariaDB databases.

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
$ stopdevel
```

# Restart development

Apache, MariaDB, DNSMasq and Mailhog can be restarted with the terminal command:

```
$ restartdev

```

# Reference

## Apache

Install Apache as service so it automatically starts at every (re)boot

```
$ brew services start httpd
```

Remove Apache service so it is no longer automatically startet at every (re)boot

```
$ brew services stop httpd
```

Start apache:

```
$ brew services start httpd
```

Stop apache:

```
$ brew services stop httpd
```

Restart apache:

```
$ brew services restart httpd
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

Edit php.ini 7.2:

```
$ code /usr/local/etc/php/7.2/php.ini
```

Edit php.ini 7.3:

```
$ code /usr/local/etc/php/7.3/php.ini
```

Edit php.ini 7.4:

```
$ code /usr/local/etc/php/7.4/php.ini
```

Edit php.ini 8.0

```
$ code /usr/local/etc/php/8.0/php.ini
```

Switch to php 5.6:

```
$ sphp 5.6
```

Switch to php 7.2:

```
$ sphp 7.2
```

Switch to php 7.3:

```
$ sphp 7.3
```

Switch to php 7.4:

```
$ sphp 7.4
```

Switch to php 8.0:

```
$ sphp 8.0
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
