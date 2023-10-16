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



