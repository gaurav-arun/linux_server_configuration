# linux_server_configuration
Udacity FSND Linux Server Configuration Project
## Goals
1. Deploy a up-to-date linux instance on the cloud.
2. Create an user `grader` with sudo permissions. Only this user should be allowed to perform remote login using ssh.
3. Secure this instance by configuring the firewall to only allow connections as follows:
  - `ssh` on `port 2200`
  - `http` on `port 80`
  - `ntp` on `port 123`
4. Deploy [CatalotApp](https://github.com/grathore07/item_catalog_app.git) project on this instance. 
5. Use Apache and mod_wsgi for handling user requests and delegating it to the app.
6. Configure and use postgresql as database server.

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

## Create a new user `grader` with sudo permissions
1. Run `sudo adduser grader`
2. Enter password as `grader`
3. Switch to user grader `su --login grader`. You will be required to enter the password.

## Give grader sudo permissions

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
2. Change `yes` to `no` if required using `PasswordAuthentication no`.
3  Run `sudo service ssh restart`

### Log in as grader from Local Machine
```
ssh -i ~/.ssh/fsnd_grader grader@<static_ip>
```
>If pop window appears, enter password as `grader`


