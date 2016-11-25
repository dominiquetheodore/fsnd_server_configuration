FSND P7 - Linux Server Configuration
nux Server Configuration

*Configuring a baseline Linux installation on a VM to securely host a web application*

# Server

 IP address: 35-164-183-31
 
 SSH Port: 2200
 
 URL: [http://ec2-35-164-183-31.us-west-2.compute.amazonaws.com/](http://ec2-35-164-183-31.us-west-2.compute.amazonaws.com/)
 
# Configuration 

## Add user grader
Added user grader with command: 

``` adduser grader ```

## Add grader to sudoers
Create a file named grader in /etc/sudoers.d and insert:

``` grader ALL=(ALL) NOPASSWD:ALL ```

## Set up SSH keys for grader

```
mkdir /home/grader/.ssh
chown grader:grader /home/grader/.ssh
chmod 700 /home/grader/.ssh
cp /root/.ssh/authorized_keys /home/grader/.ssh/
chown grader:grader /home/grader/.ssh/authorized_keys
chmod 644 /home/grader/.ssh/authorized_keys
```

```grader``` is now able to log in with ```ssh -i udacity_key.rsa grader@35.164.170.64```

## Add server to /etc/hosts
Added the following line to /etc/hosts:

```127.0.0.1 ip-10-20-33-24```

## Change timezone to UTC

Run the following command: 

``` dpkg-reconfigure tzdata ```

and select None of the above and then UTC.

## Change SSH port from 22 to 2200
Change this line in /etc/ssh/sshd_config:

``` Port 22 ```

to 

``` Port 2200 ```

Firewall was configured as per the project rubric. 

Block all incoming connections on all ports:

```sudo ufw default deny incoming```

Allow outgoing connection on all ports:

```sudo ufw default allow outgoing```

Allow incoming connection for SSH on port 2200:

```sudo ufw allow 2200/tcp```

Allow incoming connections for HTTP on port 80:

```sudo ufw allow www```

Allow incoming connection for NTP on port 123:

```sudo ufw allow ntp```

Enable the firewall:

```sudo ufw enable```

After enabling UFW, running ``` sudo ufw status ``` yields:

``` 
Status: active

To                         Action      From
--                         ------      ----
2200                       ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123                        ALLOW       Anywhere
2200 (v6)                  ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123 (v6)                   ALLOW       Anywhere (v6)
```

## Upgrade all installed packages
```
apt-get update
apt-get upgrade 
```

## Firewall
Configured the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

```
sudo ufw status

sudo ufw default deny incoming

sudo ufw default allow outgoing

```

# Software installed

Apache2 with mod_wsgi support:

```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
```

Git for version control:

```
apt-get install git
```

Flask, Postgresql and SQLAlchemy:

```
apt-get install python-flask
apt-get install python-flask-sqlalchemy
apt-get install postgresql
```

# Clone item_catalog repository to /var/www/html

Cloned the item_catalog project from Project 3 [https://github.com/dominiquetheodore/item_catalog](https://github.com/dominiquetheodore/item_catalog)  to /var/www/html:

``` git clone https://github.com/dominiquetheodore/item_catalog.git ```

The URL for the Amazon AWS instance  was also added to the list of Authorized Javascript origins and Authorized redirect URIs in Google Developers Console -> API Manager -> Credentials. 

The following downgrades were applied for the existing Google OAuth code to work:

```
pip install werkzeug==0.8.3
pip install flask==0.9
pip install Flask-Login==0.1.3
```
  
# Postgresql configuration
Created a user catalog with limited permissions to the catalog application database.

``` catalog=# CREATE USER catalog WITH PASSWORD 'XXXX' ```

Ran database_setup.py and products.py in the item_catalog folder to create and populate the database.

After logging in to the catalog database, grant the following privileges to user catalog:

``` catalog=# ALTER ROLE catalog WITH SUPERUSER ```

The item catalog app was modified to use a Postgresql database instead of sqlite as in Project 3. The following line:

``` engine = create_engine('sqlite:///catalog.db') ```

was changed to:

``` engine = create_engine('postgresql://catalog:XXXX@localhost/catalog') ```

(XXXX is the password for catalog user, not shown here for security reasons)


# Apache server configuration

Added ServerName to ``` /etc/apache2/apache2.conf ```:

``` ServerName localhost ```

The Apache2 was configured to serve the item_catalog app as a mod_wsgi application. Added the following lines to the <Virtualhost> block in /etc/apache2/sites-enabled/000-default.conf:

```
WSGIScriptAlias / /var/www/html/myapp.wsgi
<Directory /var/www/project/project/>
                    Order allow,deny
                    Allow from all
            </Directory>
        Alias /static /var/www/html/item_catalog/static
        <Directory /var/www/html/item_catalog/static/>
                    Order allow,deny
                    Allow from all
            </Directory>
```

myapp.wsgi added to item_catalog directory so that Apache2 can serve the application:

```
import sys
import os
os.chdir('/var/www/html/item_catalog')
sys.path.insert(0, '/var/www/html/item_catalog/')

from project import app as application
```

The application is now accessible from [http://ec2-35-164-183-31.us-west-2.compute.amazonaws.com/](http://ec2-35-164-183-31.us-west-2.compute.amazonaws.com/)
