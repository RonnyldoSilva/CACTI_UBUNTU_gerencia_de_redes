# How to Install and Configure Cacti on Ubuntu 18.04

![](https://github.com/RonnyldoSilva/CACTI_UBUNTU_gerencia_de_redes/blob/master/Screenshot%20from%202019-11-26%2021-46-59.png)

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
sudo gedit /etc/apache2/conf-enabled/security.conf
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
sudo ufw allow http
  
  Output:
  Rule added
  Rule added (v6
```
```
sudo ufw allow https

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

Tune MariaDB database for Cacti

Add the following settings under `[mysqld]` on the file `/etc/mysql/mariadb.cnf`:

```
max_heap_table_size=128M
tmp_table_size=128M
join_buffer_size=64M
innodb_buffer_pool_size=512M
innodb_doublewrite=OFF
innodb_flush_log_at_timeout=3
innodb_read_io_threads=32
innodb_write_io_threads=16
```

Restart mariadb service

```
MariaDB [(none)]> select @@tmp_table_size;
+------------------+
| @@tmp_table_size |
+------------------+
| 134217728 |
+------------------+
1 row in set (0.00 sec)
```

Once Database server installation is done, you need to create a database for Cacti:

```
mysql -u root -p
  
  Welcome to the MariaDB monitor. Commands end with ; or \g.
  Your MariaDB connection id is 56
  Server version: 10.3.7-MariaDB-1:10.3.7+maria~bionic-log mariadb.org binary distribution
  Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  MariaDB [(none)]> create database cacti;
  Query OK, 1 row affected (0.000 sec)
  MariaDB [(none)]> grant all privileges on cacti.* to cacti_user@'localhost' identified by 'root';
  Query OK, 0 rows affected (0.001 sec)
  MariaDB [(none)]> flush privileges; 
  Query OK, 0 rows affected (0.001 sec)
  MariaDB [(none)]> exit 
  Bye
```

Test database connection:

```
mysql -u cacti_user -p

  Enter password: 
  Welcome to the MariaDB monitor. Commands end with ; or \g.
  Your MariaDB connection id is 178
  Server version: 10.1.29-MariaDB-6 Ubuntu 18.04

  Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  MariaDB [(none)]> show databases;
  +--------------------+
  | Database |
  +--------------------+
  | cacti |
  | information_schema |
  +--------------------+
  2 rows in set (0.00 sec)

  MariaDB [(none)]>
```

### Step 4: Install SNMP and Cacti

The last package installation step is for Cacti and snmp packages. Cacti depend on Snmp and rrdtool tool for its functions. Install these packages using the command:

```
sudo apt-get install snmp snmpd snmp-mibs-downloader rrdtool cacti cacti-spine
```

When asked to select the web server, choose `Apache`.

```
+-------------------------+ Configuring cacti +--------------------------+
| Please select the web server for which Cacti should be automatically   |
| configured.                                                            |
|                                                                        |
| Select "None" if you would like to configure the web server manually.  |
|                                                                        |
| Web server:                                                            |
|                                                                        |
|                                apache2                                 |
|                                lighttpd                                |
|                                None                                    |
|                                                                        |
|                                                                        |
|                                 <Ok>                                   |
|                                                                        |
+------------------------------------------------------------------------+
```

For Database configuration, select `no` for manual configuration since we have created a database for cacti on.

```
+---------------------------+ Configuring cacti +---------------------------+
|                                                                           |
| The cacti package must have a database installed and configured before    |
| it can be used.  This can be optionally handled with dbconfig-common.     |
|                                                                           |
| If you are an advanced database administrator and know that you want to   |
| perform this configuration manually, or if your database has already      |
| been installed and configured, you should refuse this option.  Details    |
| on what needs to be done should most likely be provided in                |
| /usr/share/doc/cacti.                                                     |
|                                                                           |
| Otherwise, you should probably choose this option.                        |
|                                                                           |
| Configure database for cacti with dbconfig-common?                        |
|                                                                           |
|                    <Yes>                       <No>                       |
|                                                                           |
+---------------------------------------------------------------------------+
```

Wait for the installation to finish then proceed to configure SNMP.

### Step 5: Configure SNMP

Start by enabling the loading of MIBs by commenting out the following line on `/etc/snmp/snmp.conf`.

Change `mibs :` to `# mibs :`

Configure SNMP community name by editing `/etc/snmp/snmpd.conf`

On line 49 -  Uncomment and change to the name of community string to any name you like.

This enable full access from localhost

`rocommunity Computingforgeeks localhost`

Diable public access by commenting below lines:

```
rocommunity public default -V systemonly
rocommunity6 public default -V systemonly
```

To

```
# rocommunity public default -V systemonly
# rocommunity6 public default -V systemonly
```

Restart snmpd service

```
sudo systemctl restart snmpd
```

Validate snmp configurations using snmpwalk command line tool:

```
sudo snmpwalk -v 2c -c Computingforgeeks localhost system
SNMPv2-MIB::sysDescr.0 = STRING: Linux cacti 4.15.0-22-generic #24-Ubuntu SMP Wed May 16 12:15:17 UTC 2018 x86_64
SNMPv2-MIB::sysObjectID.0 = OID: NET-SNMP-MIB::netSnmpAgentOIDs.10
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (11034) 0:01:50.34
SNMPv2-MIB::sysContact.0 = STRING: Me <me@example.org>
SNMPv2-MIB::sysName.0 = STRING: cacti
SNMPv2-MIB::sysLocation.0 = STRING: Sitting on the Dock of the Bay
SNMPv2-MIB::sysServices.0 = INTEGER: 72
SNMPv2-MIB::sysORLastChange.0 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORID.1 = OID: SNMP-MPD-MIB::snmpMPDCompliance
SNMPv2-MIB::sysORID.2 = OID: SNMP-USER-BASED-SM-MIB::usmMIBCompliance
SNMPv2-MIB::sysORID.3 = OID: SNMP-FRAMEWORK-MIB::snmpFrameworkMIBCompliance
SNMPv2-MIB::sysORID.4 = OID: SNMPv2-MIB::snmpMIB
SNMPv2-MIB::sysORID.5 = OID: SNMP-VIEW-BASED-ACM-MIB::vacmBasicGroup
SNMPv2-MIB::sysORID.6 = OID: TCP-MIB::tcpMIB
SNMPv2-MIB::sysORID.7 = OID: IP-MIB::ip
SNMPv2-MIB::sysORID.8 = OID: UDP-MIB::udpMIB
SNMPv2-MIB::sysORID.9 = OID: SNMP-NOTIFICATION-MIB::snmpNotifyFullCompliance
SNMPv2-MIB::sysORID.10 = OID: NOTIFICATION-LOG-MIB::notificationLogMIB
SNMPv2-MIB::sysORDescr.1 = STRING: The MIB for Message Processing and Dispatching.
SNMPv2-MIB::sysORDescr.2 = STRING: The management information definitions for the SNMP User-based Security Model.
SNMPv2-MIB::sysORDescr.3 = STRING: The SNMP Management Architecture MIB.
SNMPv2-MIB::sysORDescr.4 = STRING: The MIB module for SNMPv2 entities
SNMPv2-MIB::sysORDescr.5 = STRING: View-based Access Control Model for SNMP.
SNMPv2-MIB::sysORDescr.6 = STRING: The MIB module for managing TCP implementations
SNMPv2-MIB::sysORDescr.7 = STRING: The MIB module for managing IP and ICMP implementations
SNMPv2-MIB::sysORDescr.8 = STRING: The MIB module for managing UDP implementations
SNMPv2-MIB::sysORDescr.9 = STRING: The MIB modules for managing SNMP Notification, plus filtering.
SNMPv2-MIB::sysORDescr.10 = STRING: The MIB module for logging SNMP Notifications.
SNMPv2-MIB::sysORUpTime.1 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.2 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.3 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.4 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.5 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.6 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.7 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.8 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.9 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.10 = Timeticks: (1) 0:00:00.01
```

Remember to replace `Computingforgeeks` with the name of your community string.

### Step 6: Configure Cacti Server

From here, we have to configure database settings for Cacti and initiate setup on the web interface. Change db settings on the file `/usr/share/cacti/site/include/config.php`.

```
# On line 49 - Change cacti database connection info

$database_type = "mysql";
$database_default = "cacti";
$database_hostname = "localhost";
$database_username = "cacti_user";
$database_password = "root";
$database_port = "3306";
$database_ssl = false;
```

Replace cacti_user with database user you created on Step 3 and strongpassword with cacti database user password.

Import cacti Mysql database schema:

```
mysql -u cacti_user -p cacti < /usr/share/doc/cacti/cacti.sql 

Enter password:
```

Replace cacti_user with database user and cacti with the database name.

Setup mysql timezone for cacti database user. Wait to finish it.

```
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root -p mysql

Enter password: 
Warning: Unable to load '/usr/share/zoneinfo/leap-seconds.list' as time zone. Skipping it.
```

Grant cacti MySQL database user access to TimeZone database:

```
mysql -u root -p

Enter password: 

Welcome to the MariaDB monitor. Commands end with ; or \g.
Your MariaDB connection id is 202
Server version: 10.1.29-MariaDB-6 Ubuntu 18.04

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
MariaDB [(none)]> GRANT SELECT ON mysql.time_zone_name TO cacti_user@localhost;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
```

Configure Cacti apache site Access control:

If you would like to restrict access to Cacti Web interface, edit the file `/etc/apache2/conf-available/cacti.conf` and comment out the line:

```
Require all granted
```

Then configure as below:

```
# Change line 7
#Require all granted
Require host localhost
Require ip 192.168.1.0/24
```

Replace 192.168.1.0/24 with your trusted subnet. You can also add a single IP like below:

```
Require ip 192.168.1.20
Require ip 172.16.20.30
```

You need to restart apache service after making above modifications,

```
sudo systemctl restart apache2
```

Create the follow folder:

```
sudo mkdir /opt/cacti/
```

Set directory permissions

```
sudo chown -R www-data:www-data  /opt/cacti/
```

### Step 7: Start Initial Cacti Setup
