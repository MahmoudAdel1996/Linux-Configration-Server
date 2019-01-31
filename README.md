# Linux Server Configuration
A baseline installation of Ubuntu Linux on a virtual machine to host
a Flask web application. This includes the installation of updates,
securing the system from a number of attack vectors and
installing/configuring web and database servers

## About
Ip address : 35.176.76.41   
ssh port : 2200  
passowrd of grader user : 123456

## Get Your Server
Ubuntu16.04 LTS instance on AWS [Amazon Lightsail](https://lightsail.aws.amazon.com/).


## Get Started

### Connect to your Server using SSH

* Create a directory.
        
        mkdir -p ~/.ssh

* Move the downloaded `.pem` file to `.ssh` directory we just created.
    
        mv key.pem ~/.ssh
    
        sudo chmod 400 ~/.ssh/key.pem

* Connect to instance.

        ssh -i ~/.ssh/key.pem ubuntu@35.176.76.41

### Update

        sudo apt-get update
        sudo apt-get upgrade

### Create New User grader
    
        sudo adduser gader

* Add To Superuser

        sudo usermod -aG sudo grader

* Add grader to sudo group
        
        sudo usermod -aG sudo grader

* switch to grader user

        su grader

### Create ssh key

* On local machine

        Create ssh keys using ssh-keygen, enter the name of the key e.g grader
    
* On Server

    * Create a new directory /.ssh in the grader home directory

            mkdir .ssh

    * Change .ssh permissions

            chmod 700 .ssh

    * Go to ssh directory

            cd .ssh

    * Create `authorized_keys` file on server and copy the content of the `grader.pub` from local into `authorized_keys` on server

            nano authorized_keys

    * Change `authorized_keys` permissions

            chmod 644 authorized_keys

### Edit inistance configration

go to your instance on [Lightsail](https://lightsail.aws.amazon.com/) then click on networking then Firewall Add another   
2200/tcp as custom port.

### Edit configration file

        sudo nano /etc/ssh/sshd_config

* update port to 2200
* update PermitRootLogin to be yes
* update PasswordAuthentication to be yes

### Restart ssh

        sudo service ssh restart

### connect again using command on local

        ssh grader@35.176.76.41 -p 2200 -i ~/.ssh/grader

### Configure the Firewall

        sudo ufw default deny incoming
        sudo ufw default allow outgoing
        sudo ufw allow 2200/tcp
        sudo ufw allow www
        sudo ufw allow ntp
        sudo ufw enable

### Configration Timezone

        sudo dpkg-reconfigure tzdata

* Choose none of the above
* then UTC

### Install Apache

        sudo apt install apache2

### Install Python mod_wsgi package then enable it

        sudo apt install libapache2-mod-wsgi
        sudo a2enmod wsgi

### Install postgesql

        sudo apt-get install postgresql
        sudo apt install libpq-dev python-dev postgresql postgresql-contrib

### Create DataBase

        sudo su postgres
        psql
        CREATE USER catalog WITH PASSWORD 'password';
        CREATE DATABASE catalog WITH OWNER catalog;
        \c catalog
        REVOKE ALL ON SCHEMA public FROM public;
        GRANT ALL ON SCHEMA public TO catalog;

* `CTRL + D ` 
* `CTRL + D `

### Clone the app

        sudo mkdir /var/www/
        sudo git clone https://github.com/MahmoudAdel1996/catalog.git
        sudo chown -R grader:grader /var/www/catalog
        cd /var/www/catalog

* Edit connection of database like this 

        create_engine('postgresql://catalog:123456@localhost/catalog')

* Run `models.py`

        python3 models.py

### Create wsgi file

* Go to app directory

        cd /var/www/catalog

* Create `itemscatalog.wsgi` file

        nano itemsCatalog.wsgi

* Add this content

        #!/usr/bin/python3
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/catalog/")

        from __init__ import app as application
        application.secret_key = 'secret_key'

### Configure Apache2 file

* Edit configration file 

        sudo nano /etc/apache2/sites-available/000-default.conf

* Add this content

        <VirtualHost *:80>
                ServerName servername e.g(35.176.76.41)
                ServerAdmin email e.g(hhh_hhh152001@yahoo.com)
                WSGIScriptAlias / /var/www/catalog/itemscatalog.wsgi
                <Directory /var/www/catalog/catalog/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/catalog/catalog/static
                <Directory /var/www/catalog/catalog/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>


* Restart Apache2

        sudo apache2ctl restart

### Visit your website
[Store App](http://35.176.76.41)

### created by

* Mahmoud Adel

### Resources
[postgresql connection](https://docs.sqlalchemy.org/en/latest/core/engines.html#postgresql) 

[error log file](https://unix.stackexchange.com/questions/38978/where-are-apache-file-access-logs-stored)
