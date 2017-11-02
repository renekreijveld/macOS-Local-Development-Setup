# General use

After a (re)boot of your Mac, depending on the setup Apache and MySQL are already running.
In my personal setup I have to start these manually. I open a terminal and start the services:

```
$ startdevelopment
```

Apache and MySQL are now ready for use.

# Create a new database

- Create a new database with phpMyAdmin by going in your browser to http://phpmyadmin.dev
- Click on **Databases** and below Create database enter the name of the new database and click on Create.
- The database is then ready for use. Username and password are **root**.

# Create a new website

Creating a new website is very simple. In the Sites folder create a new subfolder for your website.

As an example create a fodler **testwebsite**.

```
$ mkdir ~/Development/Sites/testwebsite
```

Then add your html/php/whatever files in the folder ~/Development/Sites/testwebsite.

There is nothing else to configure. The website is immediately ready in your browser at http://testwebsite.dev

# Stop Development

MySQL and Apache can be stopped with the terminal command:

```
$ startdevelopment
```

# Reference

## Apache

Start apache at every (re)boot:

```
$ sudo brew services start httpd
```

Stop apache at every (re)boot:

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
$ open -e /usr/local/etc/httpd/httpd.conf
```

Check if Apache is running:

```
$ ps -aef | grep httpd
```

# PHP

Edit php.ini 5.3:

```
$ open -e /usr/local/etc/php/5.3/php.ini
```

Edit php.ini 5.6:

```
$ open -e /usr/local/etc/php/5.6/php.ini
```

Edit php.ini 7.0:

```
$ open -e /usr/local/etc/php/7.0/php.ini
```

Edit php.ini 7.1:

```
$ open -e /usr/local/etc/php/7.1/php.ini
```

Change to php 5.3:

```
$ sphp 53
```

Change to php 5.6:

```
$ sphp 56
```

Change to php 7.0:

```
$ sphp 70
```

Change to php 7.1:

```
$ sphp 71
```

Change to php 7.2:

```
$ sphp 72
```

# MySQL

Start mysql at every (re)boot:

```
$ brew services start mysql
```

Stop mysql at every (re)boot:

```
$ brew services stop mysql
```

Check if MySQL is running:

```
$ ps -aef | grep mysql
```
