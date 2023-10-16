# darey.LEMPstack Project

After setting up an EC2 instance on AWS like this:

ssh into the instance and do the following:

$ sudo apt update
$ sudo apt upgrade
![sudoaptupgrade](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/sudoaptupdate&upgrade.png?raw=true)
Type Y and OK for any prompts

Then proceed to install Nginx:
$ sudo apt install nginx

Check Nginx is running:
$ systemctl status nginx

You should see 'active (running)' in green that confirms nginx is operational. 



