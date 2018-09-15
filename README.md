# Linux Server Configuration

This project is part of Udacity's [Full Stack Web Developer Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004).

**by Rafael Rios**

## About

In this project, a Linux Virtual Machine needs to be configurated to support the item catalog project.

- IP address: 34.200.235.52

- Accessible SSH port: 2200

## SSH into the VM

1. Download Private Key from the __SSH keys__ section in the __Account__ section on Amazon Lightsail.
2. Move the private key file into the folder `~/.ssh`.
3. Modify permissions of private key file with the following command

    ```chmod 400 ~/.ssh/Lightsail-key.pem```
4. SSH into the machine

    ```ssh -i ~/.ssh/Lightsail-key.pem ubunut@34.200.235.52```

## Update Packages

1. `sudo apt update`
2. `sudo apt upgrade`

## Setup grader user

1. `sudo adduser grader`
2. `sudo adduser grader sudo`

## Setup SSH

1. Configure key-based authentication for grader user

    `cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/authorized_keys`
2. Edit SSH configuration file:
    `sudo vim /etc/ssh/sshd_config`
    - Change Port 22 to Port 2200
    - Disable root login: `PermitRootLogin no`
    - Save & Quit
3. Restart SSH service:

    `sudo service sshd restart`

__Note:__ Remember to add port 2200 with TCP Protocol in the Networking section of your instance on Amazon Lightsail.

## Setup the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

- `sudo ufw allow 2200/tcp`
- `sudo ufw allow 80/tcp`
- `sudo ufw allow 123/udp`
- `sudo ufw enable`
- `sudo ufw status`

## Configure the Local Timezone to UTC

1. Configure the timezone: `sudo dpkg-reconfigure tzdata`
2. Change it to UTC

## Install Apache

  `sudo apt install apache2`

## Install mod_wsgi

- Run `sudo apt install libapache2-mod-wsgi python-dev`
- Enable mod_wsgi with `sudo a2enmod wsgi`
- Start the server with `sudo service apache2 start`

## Install and Configure PostgreSQL

- Install PostgreSQL with:

   `sudo apt-get install postgresql postgresql-contrib`
- To ensure that remote connections to PostgreSQL are not allowed, check that the configuration file `/etc/postgresql/9.3/main/pg_hba.conf` only allows connections from the local host addresses `127.0.0.1` for IPv4 and `::1` for IPv6.
- Create a PostgreSQL user called `catalog` with:

    `sudo -u postgres createuser -P catalog`
- Create an empty database called `catalog` with:

    `sudo -u postgres createdb -O catalog catalog`

## Install Git, then Clone and Setup your Item Catalog App Project

1. Install Git using `sudo apt install git`
2. Use `cd /var/www` to move to the /var/www directory
3. Create the application directory `sudo mkdir catalog`
4. Move inside this directory using `cd catalog`
5. Clone the Catalog App to the virtual machine

    `git clone https://github.com/riosdev/item-catalog.git`
6. Rename the project: `sudo mv ./item-catalog ./catalog`
7. Move to the inner catalog directory using `cd catalog`
8. Rename `project.py` to `__init__.py` using `sudo mv project.py __init__.py`.
9. Edit `database_setup.py` and `__init__.py` and change `engine = create_engine('sqlite:///catalog.db')` to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`.
10. Install pip `sudo apt install python-pip`
11. Install item catalog app dependencies:
    - `sudo pip install flask sqlalchemy flask-sqlalchemy requests`
    - `sudo pip install oauth2client psycopg2`
12. Install psycopg2 `sudo apt install python-psycopg2`
13. Create database schema `sudo python database_setup.py`
14. (Optional) Populate database

    `psql postgresql://catalog:catalog@localhost/catalog`

## Configure and Enable the Virtual Host

1. Create the catalog.conf configuration file: `sudo vim /etc/apache2/sites-available/catalog.conf`
2. Add the following lines of code to the file to configure the virtual host.

    ```
    <VirtualHost *:80>
        ServerName 34.200.235.52
        ServerAdmin r.riosdev@gmail.com
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
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
    ```
3. Enable the virtualhost with the following command: `sudo a2ensite catalog`

## Create the .wsgi File

1. Create the catalog.wsgi File under /var/www/catalog:
- `cd /var/www/catalog`
- `sudo vim flaskapp.wsgi`
2. Add the following lines of code to the catalog.wsgi file:

    ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'super_secret_key'
    ```

## Restart Apache

1. Restart Apache `sudo service apache2 restart`
