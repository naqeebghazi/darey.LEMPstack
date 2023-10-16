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


## Install PHP

You'll need the following components when installing PHP:
- PHP 
- PHP-MySQL: allows phph to communicate with mysql databases
- LibApache2-mod-PHP: enables apache to handle phph files

To install all 3 packages at once:
$ sudo apt install php libapache2-mod-php php-mysql

Check php version:
$ php -v

This finalises the LAMP stack installation

## Enable PHP on the website

We now need to set up two virtual hosts within pur EC2 instance vm.

With the default DirectoryIndex settings on Apache, a file named index.html will always take precedence over an index.php file.
This is useful for setting up maintenance pages in PHP applications, by creating a temporary index.html file containing an informative message to visitors. Because this page will take precedence over the index.php page, it will then become the landing page for the application. Once maintenance is over, the index.html is renamed or removed from the document root, bringing back the regular application page.
In case you want to change this behavior, you'll need to edit the /etc/apache2/mods-enabled/dir.conf file and change the order in which the index.php file is listed within the DirectoryIndex directive:

$ sudo vim /etc/apache2/mods-enabled/dir.conf

Then in the vim editor, ensure this is whats written (i.e. index.php is prioritised):
<IfModule mod_dir.c>
       DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>

Then, reload Apache:
$ sudo systemctl reload apache2

Optional:
_ Create a PHP script to test PHP is installed and configured on the server. 

$ sudo mkdir /var/www/projectlamp/index.php _

Next, assign ownership of the directory with your current system user:
$ sudo chown -R $USER:$USER /var/www/projectlamp

Apache on Ubuntu 20.04 has one server block enabled by default that is configured to serve documents from the /var/www/html directory.
We will leave this configuration as is and will add our own directory next next to the default one.

Then, create and open a new configuration file in Apache’s sites-available directory using your preferred command-line editor. Here, we’ll be using vi or vim (They are the same by the way):

$ sudo vim /etc/apache2/sites-available/projectlamp.conf

Paste in the following bare-bones configuration by hitting on i on the keyboard to enter the insert mode, and paste the text:

<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

![Image](https://github.com/naqeebghazi/darey.lampstack/blob/main/images/apache2sitesavailableconf.png?raw=true)

With this VirtualHost configuration, we’re telling Apache to serve projectlamp using /var/www/projectlampl as its web root directory. 
