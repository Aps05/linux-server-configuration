# Linux Server Configuration

## Server Information

- IP Address: 34.206.229.170
- SSH Port: 2200
- User: grader
- SSH Key: In submission notes
- URL: http://34.206.229.170

## Steps followed to configure the server

### Updating all packages

The first thing I did on the new host was update all installed packages.
```ssh
$ sudo apt-get update
$ sudo apt-get upgrade
```

### Setting the timezone to UTC

I used the following command:
```ssh
$ sudo dpkg-reconfigure tzdata
```
and followed the on-screen instructions to configure the timezone.

### Creating a new user

Next I created the user "grader" and gave this user sudo permission.

```ssh
$ sudo adduser grader
$ sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
$ sudo nano /etc/sudoers.d/grader
```

I changed the following line:
```
ubuntu ALL=(ALL) NOPASSWD:ALL
```
to:
```
grader ALL=(ALL) NOPASSWD:ALL
```

and hit ^O, Enter, ^X to save and exit.

### Deploying public SSH key

I generated my ssh keys on my local machine (MacOS) and then deployed the public key
on the server.

I switched to my newly created grader user:
```ssh
$ su - grader
```

I created the 'authorized_keys' file:
```ssh
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ nano .ssh/authorized_keys
```
I pasted my public key into the file and hit ^O, Enter, ^X to save and exit.

Lastly I changed the file permissions:
```ssh
$ chmod 700 .ssh
$ chmod 644 .ssh/authorize_keys
```

### Forcing Key Based Authentication

```ssh
$ sudo nano /etc/ssh/sshd_config
```

I changed the following line:
```
PasswordAuthentication yes
```
to:
```
PasswordAuthentication no
```
and hit ^O, Enter, ^X to save and exit.

I then restarted the ssh service for my change to take effect:
```ssh
$ sudo service ssh restart
```

### Changing the SSH port

```ssh
$ sudo nano /etc/ssh/sshd_config
```

I changed the following line:
```
Port 22
```
to:
```
Port 2200
```
and hit ^O, Enter, ^X to save and exit.

I then restarted the ssh service for my change to take effect:
```ssh
$ sudo service ssh restart
```

### Configuring the UFW (Uncomplicated Firewall)

Firstly I denied all incoming connections and allowed all outgoing:
```ssh
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
```

Then I allowed only the incoming ports I will be needing:
```ssh
$ sudo ufw allow 2200
$ sudo ufw allow 123
$ sudo ufw allow 80
```
And enable the firewall:
```ssh
$ sudo ufw enable
```
Entered 'y' for yes and press enter.

### Installing the necessary software

```ssh
$ sudo apt-get install apache2
$ sudo apt-get install python-setuptools libapache2-mod-wsgi
$ sudo apt-get install postgresql
```

### Creating the database and database user

I entered the postgreSQL shell from the postgre user
```ssh
$ sudo su - postgres
$ psql
```

I created the new user:
```
CREATE USER catalog;
ALTER ROLE catalog WITH PASSWORD 'grader';
```

I created the database and gave the user permission:
```
CREATE DATABASE catalog;
GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```

I quit the shell and returned to my root user:
```ssh
\p
```
```ssh
$ exit
```

### Installing git and cloning the project

```ssh
$ sudo apt-get install git
$ cd /var/www
$ mkdir catalog
$ cd catalog
$ git clone https://github.com/bonzer05/item_catalog.git
```

### Creating and populating the tables

```ssh
$ sudo python database_setup.py
$ sudo python lotsofmenus.py
```

### Creating the apache virtual host

```ssh
$ sudo touch /etc/apache2/sites-available/catalog.conf
$ sudo nano /etc/apache2/sites-available/catalog.conf
```

I pasted the following inside the file:
```
<VirtualHost *:80>
    ServerAdmin apostolis.stephanou05@gmail.com
    WSGIDaemonProcess catalog user=grader group=grader threads=5
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/static
    <Directory /var/www/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
and hit ^O, Enter, ^X to save and exit.

Then I run the following to enable the new virtual host:
```ssh
$ sudo a2ensite catalog
```

and restarted apache:
```ssh
$ sudo service apache2 restart
```

The website is now accessible at [http://34.206.229.170](http://34.206.229.170)

## Reference

I found most information that was not included in the Udacity lectures from
[DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps).
