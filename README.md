# Linux_Server_Configuration_Udacity_Project

###  Description
  baseline installation of a Linux server and prepare it to host item catalogue web application.
  
- IP address: 35.158.152.130

- Accessible SSH port: 2200

- Visit site at [http://35.158.152.130](http://35.158.152.130)

### Steps

## Create new user named grader and give sudo permission
  - Run ` sudo adduser grader`
  - Create a new file in the sudoers dir with
   `sudo nano /etc/sudoers.d/grader`
  - paste  `grader ALL=(ALL:ALL) ALL` to give access of sudo
  - Save and exit

## Update installed packages:
  - `sudo apt-get update` &  `sudo apt-get upgrade`

## Configure firewall for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable` you should see message in terminal
    "Firewall is active and enabled on system startup"

## Configure timezone to UTC
  - Run `sudo dpkg-reconfigure tzdata`
  - then choose UTC

## Change SSH port from 22 to 2200
    - Run `sudo nano /etc/ssh/sshd_config`
    - Change the port from 22 to 2200

## Configure key-based authentication for grader user
  - Run this command in your local machine `ssh-keygen`
  - run `.ssh/linuxconfig.pub`
  - copy the key until step 7

## configure key in server mahcine:
  - Run this command  `mkdir .ssh`
  - Run this command  `touch .ssh/authorized_keys`
  - Run this command to paste the generated key from your local machine to server
  `nano .ssh/authorized_keys`

## Install Apache, mod_wsgi and clone the Catalog app
  - Install apache2 `sudo apt-get install apache2`
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - run  `sudo a2enmod wsgi`
  - start apache2 server `sudo service apache2 start`
  - Install git using: `sudo apt-get install git`
  - `cd /var/www`
  - `sudo mkdir catalog`
  - `cd catalog`
  - Create a catalog.wsgi file, then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'supersecretkey'
  ```
  - Clone item catalog from github `git clone https://github.com/fouad3/Item-Catalogue_Udacity_Project catalog`
  - Rename application.py to __init__.py `mv project.py __init__.py`

## Install virtual environment,Flask and PostgreSQL :
  - Install the virtual environment `sudo pip install virtualenv`
  - Create a new virtual environment with `sudo virtualenv venv`
  - Activate the virutal environment `source venv/bin/activate`
  - Install pip with `sudo apt-get install python-pip`
  - Install Flask `pip install Flask`
  - `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`

## Update path of json files
  - `sudo nano __init__.py`
  - Change google_client_secrets.json path to `/var/www/catalog/catalog/google_client_secrets.json`  
  - Change fb_client_secrets.json path to `/var/www/catalog/catalog/fb_client_secrets.json`

## Install and configure PostgreSQL
  - `sudo apt-get install libpq-dev python-dev`
  - `sudo apt-get install postgresql postgresql-contrib`
  - `sudo su - postgres`
  - `psql`
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - Change create engine line in your `__init__.py` and `database_config.py` to:
    `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
  - `python /var/www/catalog/catalog/database_config.py`

## Configure and enable a new virtual host :
    - Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
    - Paste this code:
    ```
    <VirtualHost *:80>
      ServerName 35.158.152.130
      ServerAdmin ubuntu@35.158.152.130
      WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
      WSGIProcessGroup catalog
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
    - Enable the virtual host `sudo a2ensite catalog`

  ## Disable ssh login for root user
    - Run `sudo nano /etc/ssh/sshd_config`
    - Change `PermitRootLogin without-password` line to `PermitRootLogin no`
    - Restart ssh  `sudo service ssh restart`
    - Now you are only able to login using `ssh grader@35.158.152.130 -p 2200  -i ~/.ssh/linuxconfiguration`

  ## Restart Apache
    - `sudo service apache2 restart`
### Resources
* [Getting Started with Amazon Lightsail](https://linuxacademy.com/howtoguides/posts/show/topic/12662-getting-started-with-lightsail-a-simple-vps-solution-from-aws)
* [mod_wsgi documentation](https://modwsgi.readthedocs.io/en/develop/)
* [UFW with Fail2ban](https://askubuntu.com/questions/54771/potential-ufw-and-fail2ban-conflicts)
* [Fix locale issue](https://askubuntu.com/questions/162391/how-do-i-fix-my-locale-issue)
* [Ask Ubuntu](https://askubuntu.com/)

