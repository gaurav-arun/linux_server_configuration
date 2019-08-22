# linux_server_configuration
Udacity FSND Linux Server Configuration Project
## Goals
1. Deploy a up-to-date linux instance on the cloud.
2. Create an user `grader` with sudo permissions. Only this user should be allowed to perform remote login using ssh.
3. Secure this instance by configuring the firewall to only allow connections as follows:
  - `ssh` on `port 2200`
  - `http` on `port 80`
  - `ntp` on `port 123`
4. Use Apache and mod_wsgi for handling user http requests.
5. Configure and use postgresql as database server.
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

## Allow `grader` to log in to the cloud instance

### Steps on Local Machine
1. Run `ssh-keygen` on your local machine
2. Choose a file name for the key pair `fsnd_grader`
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








