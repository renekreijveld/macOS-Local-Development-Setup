# macOS Local Development Setup

After my MAMP PRO setup became very unstable I decided to setup my local development environment different.

I used various tips and recources from developer colleaugues.

I now run Apache, PHP (multiple versions: 5.6, 7.4, 8.0, 8.1 and 8.2) and MariaDB locally on macOS extended with Zend OPcache, APCu Cache and XDebug.
Super fast and efficient setup and all sites run with HTTPS.

Setting up a new local website was never easier: simply create a new folder in Development/Sites and the local domain https://newfolder.dev.test automatically works with SSL. There's no need to modify the apache configuation, edit your hosts file or add certificates. Everything works instantly.

This is the setup I used:

- <a href="https://github.com/renekreijveld/macOS-Local-Development-Setup/blob/master/development.setup.english.md" target="_blank">Installation</a>
- <a href="https://github.com/renekreijveld/macOS-Local-Development-Setup/blob/master/manual.english.md" target="_blank">Manual</a>
