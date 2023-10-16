# darey.LEMPstack Project

After setting up an EC2 instance on AWS, ensure inbound rules for the security group are 'All traffic', allow IPv4:
![inbound rules](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/inboundrules.png?raw=true)

ssh into the instance and do the following:

$ sudo apt update
$ sudo apt upgrade

![sudoaptupgrade](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/sudoaptupdate&upgrade.png?raw=true)

Type Y and OK for any prompts

Then proceed to install Nginx:
$ sudo apt install nginx

![nginx](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/nginx.png?raw=true)

Check Nginx is running:
$ systemctl status nginx

You should see 'active (running)' in green that confirms nginx is operational. 

![nginxsystemctlstatus](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/Screenshot%202023-10-16%20at%2021.18.56.png?raw=true)

Check that the server is reacheable locally on port 80:
$ curl -s http://localhost:80

You should see this output:
![nginx_curllocalhost](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/curllocalhost.png?raw=true)

The server will be reacheable via http at the correct address (check ec2 console)
For example: http://ec2-33-176-21-91.eu-west-2.compute.amazonaws.com/

To check public IPv4 address:
$ curl -s http://169.254.169.254/latest/meta-data/public-ipv4

## Install MySQL

$ sudo apt install mysql-server
Type Y for any prompts

Log into MySQL console:
$ sudo mysql

Once logged in, run the security script that comes with MySQL pre-installed:
$ ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
$ exit

Then in linux, start the interactive script:
$ sudo mysql_secure_installation

![Image](https://github.com/naqeebghazi/darey.lampstack/blob/main/images/MySQLserverinstallation.png?raw=true)

There is a password for logging into MySQL and a passowrd as a root user within MySQL. The prompts that follow are for both of these:

Once complete, log back into MySQL:
$ sudo mysql -p

![Image](https://github.com/naqeebghazi/darey.lampstack/blob/main/images/MySQLloginMessage.png?raw=true)

-p flag will prompt the user for the password used after changing the root user password

Notice that you need to provide a password to connect as the root user.
For increased security, it's best to have dedicated user accounts with less expansive privileges set up for every database, especially if you plan on having multiple databases hosted on your server.

[Note: At the time of this writing, the native MySQL PHP library mysqlnd doesn't support caching_sha2_authentication, the default authentication method for MySQL 8. For that reason, when creating database users for PHP applications on MySQL 8, you'll need to make sure they're configured to use mysql_native_password instead.]

Your MySQL server is now installed and secured. Next, we will install PHP, the final component in the LAMP stack.
