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

  Output: 
  PHP 7.2.5-0ubuntu0.18.04.1 (cli) (built: May 9 2018 17:21:02) ( NTS )
  Copyright (c) 1997-2018 The PHP Group
  Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
  with Zend OPcache v7.2.5-0ubuntu0.18.04.1, Copyright (c) 1999-2018, by Zend Technologies
```

Open `/etc/php/7.2/apache2/php.ini` and edit the varible to `date.timezone = "American/Recife"`.

```
sudo gedit /etc/php/7.2/apache2/php.ini
```

Ensure you set correct timezone:

```
grep date.timezone /etc/php/7.2/apache2/php.ini 

  Output:
  ; http://php.net/date.timezone
  date.timezone = "American/Recife"
```

### Step 3: Install Apache Web server

The default recommended web server for Cacti is Apache, install it using the commands:

```
sudo apt-get -y install apache2
```

After installing Apache, configure basic security by allowing Prod ServerTokens only.

```
sudo vim /etc/apache2/conf-enabled/security.conf
```

Change line 25 to `ServerTokens Prod`

This directive configures what you return as the Server HTTP response. Valid options are `Full | OS | Minimal | Minor | Major | Prod`.

Set ServerName: 

```
grep ServerName /etc/apache2/apache2.conf

  Output:
  ServerName cacti.computingforgeeks.com
```

Set Server Admin to receive an email in case of issues.

```
grep ServerAdmin /etc/apache2/apache2.conf

  Output:
  ServerAdmin adminuser@computingforgeeks.com
```

Check if you have ufw enabled, open http and https ports on the firewall.

```
ufw allow http
  
  Output:
  Rule added
  Rule added (v6
```
```
ufw allow https

  Output:
  Rule added
  Rule added (v6)
```

You need to restart apache web service after making these changes:

```
sudo systemctl restart apache2
```

### Step 3: Install MariaDB server

Before you can install MariaDB 10.4, you may need to uninstall the current version of MariaDB server. You can ignore this if upgrading.  On Ubuntu, run: 

```
sudo apt-get remove mariadb-server
```

To install MariaDB 10.4 on Ubuntu 18.04, you need to add MariaDB repository on to the system.

Step 1: Install software-properties-common if missing:

```
sudo apt-get install software-properties-common
```

Step 2: Import MariaDB gpg key:

```
sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
```

Step 3: Add the apt repository

Once the PGP key is imported, proceed to add repository URL to your Ubuntu 18.04 server: 

```
sudo add-apt-repository "deb [arch=amd64,arm64,ppc64el] http://mariadb.mirror.liquidtelecom.com/repo/10.4/ubuntu $(lsb_release -cs) main"
```

Step 4: Install MariaDB

The last step is the installation of MariaDB Server: 

```
sudo apt update
sudo apt -y install mariadb-server mariadb-client
```

You will be prompted to provide MariaDB root password. Enter a password to set.

Confirm password:

Press <Ok> to confirm the new password and install MariaDB. Make sure you note provided password.

If you were not prompted to set root password, run:

```
sudo mysql_secure_installation

  Output:
  NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
        SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

  In order to log into MariaDB to secure it, we'll need the current
  password for the root user.  If you've just installed MariaDB, and
  you haven't set the root password yet, the password will be blank,
  so you should just press enter here.

  Enter current password for root (enter for none): 
  OK, successfully used password, moving on...

  Setting the root password ensures that nobody can log into the MariaDB
  root user without the proper authorisation.

  Set root password? [Y/n] y
  New password: 
  Re-enter new password: 
  Password updated successfully!
  Reloading privilege tables..
   ... Success!


  By default, a MariaDB installation has an anonymous user, allowing anyone
  to log into MariaDB without having to have a user account created for
  them.  This is intended only for testing, and to make the installation
  go a bit smoother.  You should remove them before moving into a
  production environment.

  Remove anonymous users? [Y/n] y
   ... Success!

  Normally, root should only be allowed to connect from 'localhost'.  This
  ensures that someone cannot guess at the root password from the network.

  Disallow root login remotely? [Y/n] y
   ... Success!

  By default, MariaDB comes with a database named 'test' that anyone can
  access.  This is also intended only for testing, and should be removed
  before moving into a production environment.

  Remove test database and access to it? [Y/n] y
   - Dropping test database...
   ... Success!
   - Removing privileges on test database...
   ... Success!

  Reloading the privilege tables will ensure that all changes made so far
  will take effect immediately.

  Reload privilege tables now? [Y/n] y
   ... Success!

  Cleaning up...

  All done!  If you've completed all of the above steps, your MariaDB
  installation should now be secure.

  Thanks for using MariaDB!
```

Confirm MariaDB version: 

```
mysql -u root -p

  Output:
  Enter password: 
  Welcome to the MariaDB monitor. Commands end with ; or \g.
  Your MariaDB connection id is 49
  Server version: 10.4.6-MariaDB-1:10.4.6+maria~bionic-log mariadb.org binary distribution

  Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  MariaDB [(none)]>
```

Check version using the command:

```
MariaDB [(none)]> SELECT VERSION();

  +------------------------------------------+
  | version()                                |
  +------------------------------------------+
  | 10.4.6-MariaDB-1:10.4.6+maria~bionic-log |
  +------------------------------------------+
  1 row in set (0.000 sec)
```

