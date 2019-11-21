# How to Install and Configure Cacti on Ubuntu 18.04

## What is CACTI?

Cacti is a completely open-source network monitoring and graphing tool that was designed as a front-end application for the industry-standard data logging tool RRDtool. Cacti harness the power of RRDTool’s data storage and graphing functionality. Some good features of Cacti include:

* Fast polling of metrics
* Support for multiple data acquisition methods
* Support for advanced graph templating
* User management functionality with ACL

Cacti provide an intuitive and easy to use web interface which can be used for small LAN installations up to complex networks with thousands of servers and networking devices.

## How to Install Cacti Server on Ubuntu 18.04

Cacti has a number of dependencies that need to be installed and configured before you can deploy Cacti server itself. This guide will cover the installation of these dependencies one by one:

### Step 1: Update system and upgrade all packages

We always start with server packages upgrade to avoid any dependency issues:

```
sudo apt-get update
sudo apt-get upgrade
```

### Step 2: Install php and required modules

We now need to install php and some php modules required by cacti. Run the following commands to get everything and installed.

```
sudo apt-get -y install php php-mysql php-curl php-net-socket \
php-gd php-intl php-pear php-imap php-memcache libapache2-mod-php \
php-pspell php-recode php-tidy php-xmlrpc php-snmp \
php-mbstring php-gettext php-gmp php-json php-xml php-common
```

The most important module is php-snmp and php-mysql. Make sure they are installed. You can check your php version using the command:

```
php -v
```
Output: PHP 7.2.5-0ubuntu0.18.04.1 (cli) (built: May 9 2018 17:21:02) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
with Zend OPcache v7.2.5-0ubuntu0.18.04.1, Copyright (c) 1999-2018, by Zend Technologies

