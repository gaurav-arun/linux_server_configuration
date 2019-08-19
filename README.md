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
