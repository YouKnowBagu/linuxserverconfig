# Linux Server Config

This project will guide you through setting up a new Linux (Ubuntu) instance to serve an item catalog web application. The item catalog for my project can be found at <http://35.164.254.61/>, but will be taken offline upon completion of the Nanodegree.

# Resources

In addition to the usual Google and Stack Overflow, I also used the [Ubuntu community built wiki](https://help.ubuntu.com/community/) and the [Apache2 documentation](https://httpd.apache.org/docs/2.4/). I also used the [Filesystem Hierarchy Standard](http://www.pathname.com/fhs/pub/fhs-2.3.pdf) to further my understanding of the Linux filesystem.

# Initial Setup

Follow Udacity's provided instructions to create your new development instance, courtesy of Amazon's Elastic Cloud. Once you have your private key and public IP address, type into your terminal

    ssh -i ~/.ssh/udacity_key.rsa root@YOURPUBLICIP

where YOURPUBLICIP is the provided public IP address.

# Create new user

You will now create a new user. From the terminal, enter

    adduser grader

We will now make grader a sudo-user by adding him to the sudo group. This is accomplished by entering the following command

    sudo usermod -aG sudo grader

in -aG, the "a" stands for append, and the "G" stands for group. We are using usermod to append the group sudo to grader. There are other ways to make a sudo-user, but this is the easiest and least risky.

## Give user SSH access

On your local machine, navigate to the .ssh directory from the initial setup. We will be generating a public key for our new user. Enter

    ssh-keygen -y

You will be asked for the file in which the private key is, enter

    udacity_key.rsa

The output is the public key we will need to copy. Head back to the virtual machine and enter the following to create save the .ssh public key for user grader. If you logged out of the virtual machine, SSH back in as root and `su grader` to switch to grader.

    mkdir .ssh
    sudo nano .ssh/authorized_keys

Copy the output of `ssh-keygen` into this file, save, and exit. Change the access permissions of these files to complete ssh setup,

    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys

## Confirm access

Attempt to ssh into the server using the new user instead of root to confirm ssh access.

To confirm the user is a sudo-user, enter

    sudo ls -a /root

If you have successfully made a sudo-user you will see a list of directories, otherwise you will get a permission denied error.

# Edit sshd_config

We will now disable the root user from being able to remotely login, and change the SSH port from 22 to 2200.

Open the sshd_config file in nano (or vim, if you prefer).

    sudo nano /etc/ssh/sshd_config

Change

    # What ports, IPs and protocols we listen for
    Port 22

to

    # What ports, IPs and protocols we listen for
    Port 2200

then change the line

    #PermitRootLogin yes

to

    PermitRootLogin no

Restart sshd

    sudo /etc/init.d/sshd restart

Grader will now be our admin user, and if we need to switch to root for any reason we can do so by entering

    sudo su root

# Upgrade packages

To get a list of available packages and their versions, enter

    sudo apt-get update

To install newer versions of the packages, enter

    sudo apt-get upgrade

# Configure firewall

Continue adding security by configuring the uncomplicated firewall.  You can check the status of the firewall at any time with the command

    sudo ufw status

Begin by blocking all incoming requests.  From there we can selectively add what is needed.

    sudo ufw default deny incoming

We will enable our server to send out requests with

    sudo ufw default allow outoing

Now we will allow the incoming connections for port 2200 (SSH), port 80 (HTTP), and port 123 (NTP) and enable the firewall.

    sudo ufw allow 2200/tcp
    sudo ufw allow 80/tcp
    sudo ufw allow 123/udp
    sudo ufw enable

# Set local timezone

The local timezone should already be set for UTC, but let's be sure!  Enter

    sudo dpkg-reconfigure tzdata

Highlight "None of the above", press enter, then highlight "UTC" and press enter.  The local timezone is now configured to UTC

# Setup Apache

Time to add packages!  We will start with our HTTPServer Package, Apache2.

    sudo apt-get install apache2

Now we add the apache modules to allow the server to host Python based web applications

    sudo apt-get install python-setuptools libapache2-mod-wsgi

python-setuptools allows us to download, build, install, update, and remove Python packages.  libapache2-mod-wsgi is the WSGI adapter for Apache.  Restart apache to enable our new modules

    sudo service apache2 restart

Apache should now be ready to host our application!

# Setup PostgreSQL

In this step we will install and configure our database system PostgreSQL

    sudo apt-get install postgresql

Install Psycopg2, the database adapter

    sudo apt-get install python-psycopg2

Switch to the newly created postgres user

    sudo su postgres

And enter into the PostgreSQL

    psql

The command prompt should now look like this:

    postgres=#

We will create a new empty database named catalog

    postgres=# CREATE DATABASE catalog;

And we will create a new user named catalog

    postgres=# CREATE USER catalog;

Let's give that catalog user a password!

    postgres=# ALTER ROLE catalog with PASSWORD 'password';

And give catalog user all priveleges on the catalog database

    postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;

All finished here, quit postgreSQL

    postgres=# \q

And head back to the grader user

    exit

Database should now be up and running!

# Item catalog setup
We're going to be using git to clone our item catalog app onto our server.  Refer to the file system hierarchy at the beginning of this document for an explanation of the directories used in the next section.

First we install git on the server

    sudo apt-get install git

Move to our target directory

    cd /var/www

Clone our application repository into the directory

    git clone <https://github.com/YOURUSERNAME/YOURREPOSITORYNAME.git>

replacing YOURUSERNAME and YOURREPOSITORY name with, you guessed it, your github username and your repository's name.

We are going to need to edit several files now to configure the app to work with PostgreSQL and our server.

## Update create_engine

We will start with our database.  You will want to change the file database_setup.py.  Change the line

    engine = create_engine('sqlite:///catalog.db')

to

    engine = create_engine('postgresql://catalog:password@localhost/catalog')

## Update OAuth2

The client_secrets.json files will need to be updated for each OAuth provider you used; in my case, Google and Facebook.

### Google

Head to the API Manager for your item-catalog project in your Google developer console.  Go to credentials on the left navigation bar, and then select your catalog OAuth2 Client ID.  Under Authorized JavaScript Origins, add your Amazon Elastic cloud URI.  Use your websites public IP address in place of mine.  Be sure to note the DASHES instead of PERIODS between numbers.
<http://ec2-35-164-254-61.us-west-2.compute.amazonaws.com>

Under authorized redirect URIs, for each localhost redirect URI, add a new entry, replacing <http://localhost:8080> with <http://ec2-35-164-254-61.us-west-2.compute.amazonaws.com>  For example, if you had the following URI <http://localhost:8080/login> you would add the following entry <http://ec2-35-164-254-61.us-west-2.compute.amazonaws.com/login>

Save the changes, and download the new JSON file.  Copy the contents and paste them into the JSON file on your server.

### Facebook

Navigate to your application on the Facebook developers site.  Go to the Facebook Login section under Products, go to Settings, and add <http://ec2-35-164-254-61.us-west-2.compute.amazonaws.com/> to the OAuth redirect URIs.  For Facebook, you do not need to update the JSON files.

# Configure Virtual Host

Refer to the [Apache Documentation](https://httpd.apache.org/docs/2.4/) for more information on this step.  We are going to tell Apache how to handle requests, where to find site files, etc.  To start, create and open the configuration file for your app

    sudo nano /etc/apache2/sites-enabled/000-default.conf

The file should look like this

    <VirtualHost \*:80>

        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin YOUREMAIL
        DocumentRoot /var/www/html
        WSGIScriptAlias / /var/www/item-catalog/app.wsgi

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
    </VirtualHost>

# Create and configure wsgi files

The finish line is in site (get it?)!  We now need to make the wsgi file inside of our applications directory.

    sudo nano /var/www/YOURAPPDIRECTORYNAME/app.wsgi

and in this file, paste the following

    \#!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/item-catalog/")
    from app import app as application
    application.secret_key = 'super_secret_key'

# Install required Python packages

Start by installing the Python package Manager

    sudo apt-get install python-pip

Using the python package manager, install the pipreqs package

    sudo pip install pipreqs

using pipreqs, create a requirements.txt file

    sudo pipreqs /var/www/item-catalog/app

and finally, using pip, install all requirements from the requirements.txt file

    sudo pip install requirements.txt

# Restart and enjoy!

    sudo service apache2 restart

Your site should now be available through your public IP address.  Congrats!
