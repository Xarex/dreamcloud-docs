=====================================================
Step-by-step guide to deploy ownCloud on DreamCompute
=====================================================

Preparation
~~~~~~~~~~~

This tutorial details the installation of ownCloud on a DreamCompute
instance.  We'll install and configure all necessary components without making
use of automatic configuration management systems.

First, deploy a Ubuntu 16.04 LTS virtual machine.  It is recommended to boot a
volume backed instance as they are permanent as opposed to ephemeral disks and
can be larger than 80GB in size if larger amounts of data will be stored.  This
can be done in the `web UI <215912848>`_ or the `nova client <215912778>`_.

Optionally, multiple instances can be used such as one hosting ownCloud HTTP
and a second hosting the database.  It is strongly recommended to use private
networking for such a setup.  The other differences include the MySQL instance
having a security group rule to open port 3306, MySQL listening on the correct
IP address, and MySQL user allowing other than localhost.

To start installing software, login to your DreamCompute instance:

.. code-block:: console

    [user@localhost]$ ssh ubuntu@$IP

changing the IP to your server's public IP address.

Installing MariaDB
~~~~~~~~~~~~~~~~~~

To install MariaDB, run:

.. code-block:: console

    [user@server]$ sudo apt-get update
    [user@server]$ sudo apt-get install mariadb-server

During installation the root database user will be setup without a password,
however due to an authentication plugin will prevent login from anyone but
the operating system root user.  A password can be set if desired, otherwise
MySQL root access is gained like so:

.. code-block:: console

    [user@server]$ sudo mysql -u root
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 55
    Server version: 10.0.29-MariaDB-0ubuntu0.16.04.1 Ubuntu 16.04

    Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    MariaDB [(none)]>

Configuring MariaDB
~~~~~~~~~~~~~~~~~~~

By default the database server will listen only on localhost, not exposing the
MySQL server to the outside world.  This is optimal for security, but can be
equally as secure as a separate instance with private networking.

Add a MySQL user for ownCloud
-----------------------------

It is best practice to make a new MySQL user for use with ownCloud for security
purposes.  To do this, connect to MySQL:

.. code-block:: console

    [user@server]$ sudo mysql -u root

and run the following:

.. code-block:: sql

    CREATE DATABASE owncloud;
    CREATE USER 'owncloud'@'localhost' IDENTIFIED BY 'PASSWORD';
    GRANT ALL on owncloud.* to 'owncloud'@'localhost';
    flush privileges;
    exit

where **PASSWORD** is the desired password for the ownCloud software.  This
example uses 'owncloud' as the MySQL user name and database name for
simplicity.

Installing the ownCloud application
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Installing Dependencies
-----------------------

Now that we have a database that ownCloud can use, we need to deploy the
frontend application.  To do this run:

.. code-block:: console

    [user@server]$ sudo apt-get install apache2 libapache2-mod-php php-gd \
                   php-json php-mysql php-curl php-intl php-mcrypt \
                   php-imagick php-zip php-dom php-mbstring

to install the packages that ownCloud requires to run.

Downloading ownCloud
--------------------

Now we need to download the actual ownCloud application. Do this by going to
https://owncloud.org/install/#instructions-server in a browser and right click
the *.tar.bz2* link and click *copy link location* then run:

.. code-block:: console

    [user@server]$ wget $URL

where **$URL** is the URL you just copied. This will download a compressed
copy of the ownCloud application. Decompress the file by running:

.. code-block:: console

    [user@server]$ tar xvf owncloud-9.1.4.tar.bz2

The version numbers for your download might be different than the above, so
adjust as necessary.  This will create a directory called "owncloud" in the
current directory.

Setting up the owncloud directory
---------------------------------

Next, copy the owncloud directory to the correct location.  In this guide, it
will be running it at /var/www/owncloud. To copy it run:

.. code-block:: console

    [user@server]$ sudo mv /home/ubuntu/owncloud /var/www/

Now we want to change the permissions of the owncloud directory so that the web
user, www-data in our case, can access it. Do this by running

.. code-block:: console

    [user@server]$ sudo chown -R www-data:www-data /var/www/owncloud

If the ownCloud package is no longer needed, clean it up by running:

.. code-block:: console

    [user@server]$ rm owncloud-9.1.4.tar.bz2

As before, the file name may vary with different versions so adjust the
command as needed.

Configuring Apache
------------------

Now that ownCloud is in the right place, configure Apache to use it. To do
this, create the file /etc/apache2/sites-available/owncloud.conf with the
following command:

.. code-block:: console

    [user@server]$ sudo bash -c 'cat > /etc/apache2/sites-available/owncloud.conf << "EOF"
    Alias /owncloud "/var/www/owncloud/"

    <Directory /var/www/owncloud/>
      Options +FollowSymlinks
      AllowOverride All

     <IfModule mod_dav.c>
      Dav off
     </IfModule>

     SetEnv HOME /var/www/owncloud
     SetEnv HTTP_HOME /var/www/owncloud

    </Directory>
    EOF'

To enable this new config, enable this new configuration by running:

.. code-block:: console

    [user@server]$ sudo a2ensite owncloud

Next, enable an apache module needed for ownCloud by running:

.. code-block:: console

    [user@server]$ sudo a2enmod rewrite

You should also use SSL with ownCloud to protect login information and data.
Apache installed on Ubuntu comes with a self-signed cert. To enable SSL using
that cert run:

.. code-block:: console

    [user@server]$ sudo a2enmod ssl
    [user@server]$ sudo a2ensite default-ssl
    [user@server]$ sudo service apache2 restart

Finishing the Installation
~~~~~~~~~~~~~~~~~~~~~~~~~~

Now everything is configured on the server.  Open a browser and visit
https://IP/owncloud where **IP** is the IP address of your instance.  The
website will ask for a username and password, a data storage location which can
be kept as the default, and then the database information.  The username and
database name are 'owncloud' unless modified from the above, the host can
remain 'localhost' and the password used can be entered.

Click to continue, and if all is setup correctly the ownCloud files page will
load.  Congratulations on your new ownCloud install!

.. meta::
    :labels: owncloud
