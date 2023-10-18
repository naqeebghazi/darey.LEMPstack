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

## Install PHP

You have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server.

While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. You’ll need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

To install these 2 packages at once, run:

$ sudo apt install php-fpm php-mysql
When prompted, type Y and press ENTER to confirm installation.

You now have your PHP components installed.

## Configure nginx to use PHP

Create the root web directory for your_domain as follows:

$ sudo mkdir /var/www/projectLEMP

Assign ownership of the directory with the $USER environment variable, which will reference your current system user:

$ sudo chown -R $USER:$USER /var/www/projectLEMP

Open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:

$ sudo nano /etc/nginx/sites-available/projectLEMP

Paste this into the new file:

#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}

![etcnginx_serverCode](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/etcnginx_serverCode.png?raw=true)


Here’s what each of these directives and location blocks do:

listen — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.
root — Defines the document root where the files served by this website are stored.
index— Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.
server_name — Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.
location / — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.
location ~ \.php$ — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.
location ~ /\.ht— The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root ,they will not be served to visitors.
When you’re done editing, save and close the file. If you’re using nano, you can do so by typing CTRL+X and then y and ENTER to confirm.

Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:

sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:

$ sudo nginx -t
You shall see following message:

    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful

If any errors are reported, go back to your configuration file to review its contents before continuing.

We also need to disable default Nginx host that is currently configured to listen on port 80, for this run the following. This may not be necessary if the default file is not present. 

$ sudo unlink /etc/nginx/sites-enabled/default

When you are ready, reload Nginx to apply the changes:

$ sudo systemctl reload nginx

![nginxcomplete](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/nginxcomplete.png?raw=true)

Your new website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that your new server block works as expected:

$ sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html

If you get a permission denied like below, ensure the projectLEMP directory does not have an index.html file and then run the command again.

![sudoechoindex](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/sudoechoIndexhtml.png?raw=true)

Now go to your browser and try to open your website URL using IP address:

http://<Public-IP-Address>:80
If you see the text from ‘echo’ command you wrote to index.html file, then it means your Nginx site is working as expected.
In the output you will see your server’s public hostname (DNS name) and public IP address. You can also access your website in your browser by public DNS name, not only by IP – try it out, the result must be the same (port is optional)
![browserURLnginx](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/browserURLnginx.png?raw=true)

http://<Public-DNS-Name>:80
You can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it. Once you do that, remember to remove or rename the index.html file from your document root, as it would take precedence over an index.php file by default.

![DNSbrowser](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/DNSbrowser.png?raw=true)

Your LEMP stack is now fully configured. In the next step, we’ll create a PHP script to test that Nginx is in fact able to handle .php files within your newly configured website.

## Testing PHP with Nginx

LEMP stack is now fully set up. 

To test/validate that nginx can corrcetly handle .php files do this:

- Use vim to create a info.php file in the /var/www/projectLEMP/ directory.
- Paste into the info.php file:
    <?php
    phpinfo();

- Access this php file in your web browser:
    http://`server_domain_or_IP`/info.php

You will see this page with detailed information about your server, thus demonstrating a successful test:

![phpPagebrowser](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/phpPagenginx.png?raw=true)

Once confirmed, delete the info.php file as keeping it poses a security risk due to the server information it has on it. Use this command:

$ sudo rm /var/www/your_domain/info.php
