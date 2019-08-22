# linux_server_configuration
Udacity FSND Linux Server Configuration Project

## Deployed Item Catalog App
- Open [catalogapp.ddns.net](http://catalogapp.ddns.net). Login using your google account.
- You can also use the static ip ([http://13.234.235.47](http://13.234.235.47)) to reach the web application. But google login will not work with this approach.

## Goals
1. Deploy a up-to-date linux instance on the cloud.
2. Create an user `grader` with sudo permissions. Only this user should be allowed to perform remote login using ssh.
3. Secure this instance by configuring the firewall to only allow connections as follows:
  - `ssh` on `port 2200`
  - `http` on `port 80`
  - `ntp` on `port 123`
4. Configure and use postgresql as database server for Item Catalog App.
5. Use Apache and mod_wsgi for handling user http requests.
6. Deploy [CatalotApp](https://github.com/grathore07/item_catalog_app.git) project to use WSGI on cloud instance.

### Create Linux instance on Amazon Lightsail
1. Create an amazon lightsail account [here](https://lightsail.aws.amazon.com)
2. Click on `Create Instance`
3. Choose `Linux/Unix`, `OS Only` and `Ubuntu 16.04LTS`.
4. Provide instance name as `udacity_fsnd`
5. Click on `Create Instance` on the bottom.
6. It takes a few minutes for the instance to be provisioned.
7. Click '**Download default key**'
8. A file with extension `.pem` will be downloaded. Rename this file to `udacity_fsnd.rsa` and save it in ~/.ssh directory.
9. Click on the instance and assign it a static ip by clicking on `Static IP` and following the instructions. This ensures that Stopping and Starting the instance does not change its IP.

### First Login into linux instance
1. Run `chmod 600 ~/.ssh/udacity_fsnd.rsa` to restrict the permissions.
2. ssh into the linux instance using `ssh -i ~/.ssh/lightrail_key.rsa ubuntu@<your_static_ip>`
3. Run `sudo apt-get update` to download and update the package lists for all the repositories and PPAs.
4. Run `sudo apt-get upgrade` to upgrade all the packages and their dependencies to the newest version.

## Create a new user `grader` with sudo permissions
1. Run `sudo adduser grader`
2. Enter password as `grader`
3. Switch to user grader `su --login grader`. You will be required to enter the password.

## Give `grader` sudo permissions

1. Go back to the default user `ubuntu` by typing `exit`
2. Create a file `grader` in `/etc/sudoers.d` using  `sudo touch /etc/sudoers.d/grader`
3. Open this file `sudo nano /etc/sudoers.d/grader`
4. Add line `grader ALL=(ALL:ALL) ALL` to this file. Save and exit.
5. Verify that grader has sudo permissions.
```
su --login grader
Enter password:
sudo -l

Matching Defaults entries for grader on ip-172-26-6-175.ap-south-1.compute.internal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User grader may run the following commands on ip-172-26-6-175.ap-south-1.compute.internal:
    (ALL : ALL) ALL
```

## Enable remote login to the cloud instance as `grader`
We need to generate a ssh key pair on the local machine and add the public key to the `~/.ssh/authorized_keys` for `grader` on cloud instance.

### Steps on Local Machine
1. Run `ssh-keygen` on your local machine
2. Choose a file name for the key pair - `fsnd_grader`
3. Enter passphrase as `grader`
4. Two files will be created in your local machine `~/.ssh` directory : `fsnd_grader` and `fsnd_grader.pub`
5. Install xclip on your local machine using `sudo apt-get install xclip`
6. Copy the contents of `fsnd_grader.pub` to clipboard with `xclip -sel clip < ~/.ssh/id_rsa.pub`

### Steps on Cloud Instance
1. Log in to the cloud instance.
2. Switch to `grader` home directory(`~`), and create a new `.ssh` directory `mkdir .ssh`
3. Run `nano .ssh/authorized_keys` 
4. Paste the contents from the clipboard using right click.
5. Set correct permission for .ssh directory using `chmod 700 .ssh`
6. Set correct permission for authorized_keys using `chmod 644 .ssh/authorized_keys`

### Disable tunnelled clear text passwords on Cloud Instance
1. Run `nano /etc/ssh/sshd_config` and search for keyword `PasswordAuthentication`
2. Change `yes` to `no` if required, by changing line to `PasswordAuthentication no`
3. Run `sudo service ssh restart` for the changes to take effect.

### Log in as `grader` from Local Machine
```
ssh -i ~/.ssh/fsnd_grader grader@<static_ip>
```
>If pop window appears, enter password as `grader`

## Configure `ufw` firewall for Cloud Instance
1. First step is to disable default ssh port 22 and configure ssh server to use port 2200. Open file `sudo nano /etc/ssh/sshd_config`, **change the port number on line 5 to 2200**, then **restart SSH** by running `sudo service ssh restart`

2. Check the status of ufw using `sudo ufw status`. It should show `inactive`.
3. Use the following sequence of command to configure the firewall
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw deny 22
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw enable
```
Firstly, we deny all incoming connection and allow all outgoing connections. We block `port 22` for default ssh connection and configure `port 2200` to allow tcp traffic. Since we have already configured ssh to use port 2200, all ssh connection will happen on this port going forward. We allow http traffic on default `port 80` and udp traffic on `port 123` for ntp.

Finally, we put all this ufw configuration in effect by enabling ufw.

Run `sudo ufw status` to verify. It is configured for both IPv4 and IPv6.
```
To                         Action      From
--                         ------      ----
22                         DENY        Anywhere                  
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
22 (v6)                    DENY        Anywhere (v6)             
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)  
```

### (OPTIONAL) Verify open ports on Cloud instance from the Local Machine
```
sudo nmap -Pn <static_ip>

Starting Nmap 7.01 ( https://nmap.org ) at 2019-08-20 01:36 IST
Nmap scan report for ec2-13-234-235-47.ap-south-1.compute.amazonaws.com (13.234.235.47)
Host is up (0.026s latency).
Not shown: 998 filtered ports
PORT     STATE SERVICE
80/tcp   open  http
2200/tcp open  ici

Nmap done: 1 IP address (1 host up) scanned in 8.26 seconds
```

```
sudo nmap  -sU -T4 -Pn <static_ip>

Host is up (0.023s latency).
Not shown: 999 open|filtered ports
PORT    STATE SERVICE
123/udp open  ntp
```

## Install and Configure `postgresql` 
1. Run `sudo apt-get install postgresql`.

### Create a new Linux user called `catalogapp`

1. Create a new Linux user `catalogapp` and provide sudo permissions following the same steps as we did for `grader`
2. Verify that this new user has been created.
```
grader@ip-172-26-6-175:~$ su --login catalogapp
Password: 
catalogapp@ip-172-26-6-175:~$ sudo -l
[sudo] password for catalogapp: 
Matching Defaults entries for catalogapp on ip-172-26-6-175.ap-south-1.compute.internal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User catalogapp may run the following commands on ip-172-26-6-175.ap-south-1.compute.internal:
    (ALL : ALL) ALL
```

### Configure `postgresql` as database server
1. Switch to default postgresql user using `sudo -u postgres psql`. Postgres prompt would appear.
2. Create a new postgresql user for connecting to the database from the `CatalogApp` as follows:
```
CREATE ROLE catalogapp WITH LOGIN;
ALTER ROLE catalog CREATEDB;
\password catalogapp
\du
```

First we create a `catalogapp` user with ability to create databases. Then we set a password for this user(Here it is set to `catalogapp`). Then we check the list of users to verify. We will see the following output.

```
                                    List of roles
 Role name  |                         Attributes                         | Member of 
------------+------------------------------------------------------------+-----------
 catalogapp | Create DB                                                  | {}
 postgres   | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

```
3. Create a database called `catalogapp` by running `createdb catalogapp`
4. Run `\l` to see that the new database has been created.
5. Finally we can exit out of postgresql prompt using `\q`.

## Install git and python3
1. `sudo apt-get install git`
2. `sudo apt-get install python3`

## Configure Item Catalog app to use `postgresql` as database server
1. Chage directory to `/var/www` using `cd /var/www`

2. Clone [CatalogApp](https://github.com/grathore07/item_catalog_app) using `sudo git clone https://github.com/grathore07/item_catalog_app`

3. Change the following line in all the files that try to connect to database. In my case I changed this line in `main.py`, `database_setup.py` and `db_bootstrap.py`. 
```
+engine = create_engine("postgresql://catalogapp:catalogapp@localhost/catalogapp")
```
Here `catalogapp:catalogapp` is the postgresql `username` and `password` that we created earlier and last `catalogapp`
is the name of the database to connect to.

4. cd into `/var/www/item_catalog_app` directory. Create a virtual environment `item_catalog_env` for installing all the required python packages and activate it.

```
sudo pip3 install virtualenv=16.0.0
virtualenv item_catalog_env
source item_catalog_env/bin/activate
```

5. Once the `item_catalog_env` is activated, install all python packages required for running the project using `requirements.txt` file.

```
pip3 install -r requirements.txt
```

6. Test run the app to confirm that the app is launching and is able to use `postgresql` as database.
```
python3 database_setup.py
python3 db_bootstrap.py
python3 main.py
```
Executing `database_setup.py` connects to postgresql and creates the `User` and `Item` tables in the `catalogapp` database.
Then `db_bootstrap.py` populates the database with random items and users. Finally running `main.py` should launch the app without any errors. **After verifying, follow steps 7 and 8 to make wsgi configuration easier.**

7. Move `main.py` to `__init__.py`.
```
sudo rm __init__.py
sudo mv main.py __init__.py
```
8. Change line `app.run('0.0.0.0', port=5000)` to `app.run()` in `__init__.py` file.

## Install `Apache`
1. `sudo apt-get install apache2`
2. Enter the `static_ip` in your web browser to verify that apache is installed. We should be able to see default apache web page.
3. After verifying, run `sudo a2dissite 000-default.conf` to disable apache site.
4. Restart the apache server with `sudo service apache2 reload`

## Install `mod_wsgi`
1. `sudo apt-get install libapache2-mod-wsgi-py3 python3-dev`
2. Enable mod_wsgi by running `sudo a2enmod wsgi`


### Setup Apache service
1. Create a configuration file `item_catalog_app.conf` in `/etc/apache2/sites-available/` with the following content:
```
<VirtualHost *:80>
                Servername <static_ip>
                ServerAdmin <your@email.id>
                WSGIScriptAlias / /var/www/item_catalog_app.wsgi
                <Directory /var/www/item_catalog_app/>
                        Order allow,deny
                        Allow from all
                        Options -Indexes
                </Directory>
                Alias /static /var/www/item_catalog_app/static
                <Directory /var/www/item_catalog_app/static/>
                        Order allow,deny
                        Allow from all
                        Options -Indexes
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Here `static_ip` is our static ip address and `your@email.id` is our email address.

2. Run `sudo a2ensite item_catalog_app` to enable apache server.

3. Run `sudo service apache2 reload` to restart apache server


### Setup mod_wsgi as WSGI for Item Catalog app
1. cd into `/var/www`
2. Create a wsgi file using `sudo nano item_catalog_app.wsgi`.
3. Add following lines to wsgi file
```
# Activate virtual environment for running the app
python_home = '/var/www/item_catalog_app/item_catalog_env'
activate_this = python_home + '/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

import logging
import os
import sys

logging.basicConfig(stream=sys.stderr)
os.chdir('/var/www')


# Add path to the application so that various files
# can be found by python.
sys.path.insert(0, "/var/www/item_catalog_app/")
sys.path.insert(0, "/var/www/")

# Create app instance and let mod_wsgi handle it.
from item_catalog_app import app as application
application.secret_key = 'supersecretkey'
```

This file activates the `item_catalog_env` first. Then it configures the logger. Finally, it imports `item_catalog_app` as a python package which in turn instantiates the flask app and mod_wsgi takes over from this point.

## Configure Google OAuth (OPTIONAL)
Google OAuth requires you to provide a list  of `Authorized Javascript Origin` and `Authorized redirect URIs` to avoid CORS vulnerability. Follow these steps:
1. Attach a domain name to your `static_ip` using free dynamic dns service like `www.no-ip.com`. Copy this domain name.
2. Log in to your google developer console. Find the web client you have configured for Item Catalog app.
3. Paste the domain name to both `Authorized Javascript Origin` and `Authorized redirect URIs`.
4. Save your changes.
