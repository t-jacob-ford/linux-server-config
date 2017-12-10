# Catalog App
## Table of Contents
- IP Address
- URL
- Summary of Installed Software
- Summary of Configurations Made
- Third Party Resources

## IP Address
18.221.222.59

## URL
http://18.221.222.59

## Summary of Installed Software
| Package | README |
| ------ | ------ |
| Apache2 | [https://httpd.apache.org] |
| libapache2-mod-wsgi | [https://packages.debian.org/wheezy/libapache2-mod-wsgi] |
| python-flask | [http://flask.pocoo.org] |
| postgresql | [https://www.postgresql.org/] |
| postgresql-contrib | [https://packages.debian.org/jessie/postgresql-contrib-9.4] |


## Summary of Configurations Made
*Via AWS web interface*
Allow a custom incoming TCP connection on port 2200. 
- Once you complete the entire configuration you should close the standard port connection through this AWS web interface, disabling SSH over port 22. 

### Update all installed packages
*Via SSH connection to AWS - Ubuntu Instance*
`$ sudo apt-get update`
`$ sudo apt-get upgrade`

### Create new user and give sudo permission 
`$ adduser grader`
`$ sudo visudo`
        
        grader ALL=(ALL:ALL) ALL

### Edit the SSH configuration
`$ sudo vi /etc/ssh/sshd_config`

        Port 2200
        PermitRootLogin no
        PasswordAuthentication yes
        >> AllowUsers grader
Save the midified sshd_config file and restart SSH with `$ /etc/init.d/ssh/restart`
### Generate SSH keys, push to server, and authenticate with password
*Via Bash shell on client machine*
`$ ssh-keygen`
`$ ssh-copy-id grader@18.221.222.59 -p2200`
Now edit sshd_config to deny PasswordAuthentication, and properly protect your ssh key on your local machine.
### Configure Firewall
*Via SSH connection to AWS - Ubuntu Instance*
`$ sudo ufw allow 2200/tcp`
`$ sudo ufw allow 123/udp`
`$ sudo ufw allow 80/tcp`
`$ sudo ufw enable`
### Install Apache2 and Flask
` $ sudo apt-get install apache2`
`$ sudo apt-get update`
`$ sudo apt-get install libapache2-mod-wsgi`
`$ sudo apt-get install python-flask`
`$ sudo apt-get install upgrade`

### Create Directory structure for the app
`$ sudo mkdir /var/www/FlaskApps`
`$ sudo mkdir /var/www/FlaskApps/catalog`
`$ sudo mkdir /var/www/FlaskApps/catalog/static`
`$ sudo mkdir /var/www/FlaskApps/catalog/templates`

Now push your project files from your local client to the appropriate dir under /var/www/FlaskApps with `scp`.

Example: `$ scp -P 2200 -i grad-key /C/Users/dej/catalog/* grader@18.221.222.59:/var/www/FlaskApps`
### Create Configuration file 
*This serves as a pointer to the Flask app/site*
`$ sudo vi /etc/apache2/sites-available/catalog.config`
    
    <VirtualHost *:80>
        ServerName 18.221.222.59
        ServerAdmin admin@mywebsite.com
        WSGIScriptAlias / /var/www/FlaskApps/FlaskApps.wsgi
        <Directory /var/www/FlaskApps/CatalogApp/>
            Order allow,deny
            Allow from all
        </Directory>
        <Directory /var/www/FlaskApps/CatalogApp/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
Save the modified text and quit.
### Create Web Server Gateway Interface file 
*This serves as instructions to tell Apache how to run flask*
`$ sudo vi /var/www/FlaskApps/FlaskApps.wsgi`

    #! /usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/FlaskApps/CatalogApp/")

    # home points to the home.py file
    from home import app as application
    application.secret_key = "somesecretsessionkey"

Save the modified text and quit.
### Restart WSGI and Apache services

    $ sudo a2enmod wsgi
    $ sudo apachectl restart
    $ sudo a2ensite PlagiarismDefenderApp
    $ sudo service apache2 restart
    $ sudo /etc/init.d/apache2 reload
### Create and Configure PostgreSQL database
`$ sudo apt-get install postgresql postgresql-contrib`

1- Create new user specifically for psql configuration.
2- Switch to the new user and connect to the system

    # CREATE USER catalog WITH PASSWORD 'PW-FOR-DB';
    # ALTER USER catalog CREATEDB;
    # CREATE DATABASE catalog WITH OWNER catalog;
    # REVOKE ALL ON SCHEMA public FROM public;
    # GRANT ALL ON SCHEMA public TO catalog;
quit and exit
### Modify Flask Application to use psql engine
 - For this specific application the three files: db_setup.py / populate_db.py / and catalog.py need to be modified.

The lines previously reading:

    engine = create_engine('sqlite:///vws.db')
    
Should be changed to:

    engine = create_engine('postgresql://catalog:852852@18.221.222.59/catalog')

### Run Application
`$ sudo service apache2 restart`

Navigate in browser to http://18.221.222.59
### Error checking and Debugging
`$ sudo tail -15 /var/log/apache2/error.log`

### Third-Party Resources
Udacity Instructors
Flask:
http://flask.pocoo.org
Apache2:
https://httpd.apache.org
Digital Ocean:
https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps
Killtheyak:
http://killtheyak.com/use-postgresql-with-django-flask/
Manuel Amunategui:
https://github.com/amunategui/amunategui.github.io
Stuken(GitHub):
https://github.com/stueken/
