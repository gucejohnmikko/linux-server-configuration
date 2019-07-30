Linux Server Configuration Project

About the project
- A baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

In choosing the server:
-Start a new Ubuntu Linux server instance on Amazon Lightsail. There are full details on setting up your Lightsail instance on the link provided [udacity_project_details](https://classroom.udacity.com/nanodegrees/nd004/parts/b2de4bd4-ef07-45b1-9f49-0e51e8f1336e/modules/56cf3482-b006-455c-8acd-26b37b6458d2/lessons/046c35ef-5bd2-4b56-83ba-a8143876165e/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e.)

Steps to Configure the server
ssudsu
1. Update all packages
-sudo apt-get update
-sudo apt-get upgrade
-sudo apt-get autoremovesu


2. Change timezone to UTC 
-sudo dpkg-reconfigure tzdata
-select None of the Above
-select UTC

3. Create a new user grader and Give him sudo access
-sudo adduser grader
-sudo adduser grader sudo

or
-sudo nano /etc/sudoers.d/grader 
-then add the following text grader ALL=(ALL) ALL

4. Setup SSH keys for grader
-On local machine ssh-keygen Then choose the path for storing public and private keys
-On remote machine home as user grader
-sudo su - grader
-mkdir .ssh
-touch .ssh/authorized_keys 
-sudo chmod 700 .ssh
-sudo chmod 600 .ssh/authorized_keys 
-nano .ssh/authorized_keys 
-Then paste the contents of the public key created on the local machine

5. Change the followin in sshd_config for root user
-sudo nano /etc/ssh/sshd_config
-Change the following:
    1. Find the Port line and edit it to 2200.
    2. Find the PermitRootLogin line and edit it to no.
    3. Find the PasswordAuthentication line and edit it to no.
-Save the file and run sudo service ssh restart

6. Configure the Uncomplicated Firewall (UFW)
-sudo ufw default deny incoming
-sudo ufw default allow outgoing
-sudo ufw allow 2200/tcp
-sudo ufw allow www
-sudo ufw allow ntp
-sudo ufw enable

7. Install Apache2 and mod-wsgi for python3 and Git
-sudo apt-get install apache2 libapache2-mod-wsgi-py3 git(from project details)
-Note: For Python2 replace libapache2-mod-wsgi-py3 with libapache2-mod-wsgi (since my catalog is on python2)

8. Install and configure PostgreSQL
-sudo apt-get install libpq-dev python-dev
-sudo apt-get install postgresql postgresql-contrib
-sudo su - postgres
-psql


9. Create a new database user named catalog that has limited permissions to your catalog application database.
-CREATE USER catalog WITH PASSWORD 'password';
-CREATE DATABASE catalog WITH OWNER catalog;
-\c catalog
-REVOKE ALL ON SCHEMA public FROM public;
-GRANT ALL ON SCHEMA public TO catalog;
-\q
-exit

Note: In your catalog project you should change database engine to
engine = create_engine('postgresql://catalog:password@localhost/catalog')

10. Clone the Catalog app from GitHub and Configure it
-cd /var/www/
-sudo mkdir catalog
-sudo chown grader:grader catalog
-git clone <your_repo_url> catalog
-cd catalog
-nano catalog.wsgi

-Then add the following in catalog.wsgi file

    #!/usr/bin/python
    import sys
    import logging  
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'super_secret_key'


11. Configure apache server
sudo nano /etc/apache2/sites-enabled/000-default.conf
Then add the following content:

    <VirtualHost *:80>
     ServerName ec2-54-191-54-68.compute-1.amazonaws.com
     ServerAdmin gucejohnmikko@gmail.com
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


12. Reload & Restart Apache Server
-sudo service apache2 reload
-sudo service apache2 restart

Resources
Amazon Lightsail
[digitalocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
