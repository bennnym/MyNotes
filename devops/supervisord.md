## Steps to install Supervisor

Some of these steps may not be required:
1. `sudo easy_install supervisor`
2. `sudo amazon-linux-extras install epel`
3. `sudo yum install supervisor` - THIS IS REQUIRED

Now we navigate to the supervisor config file and alter it slightly so that it knows to look for our custom app.config file where we are going to store the specific commands to execute.

The config file is located in the root at `/etc`

Now alter the supervisord.conf by `sudo vim supervisord.conf` and scroll right to the bottom and add in where you are going to place your `app.conf` file. In our case that is here:
```csharp
[include]
files = supervisord.d/app.conf
```

Now we navigate to the `supervisord.d` folder and add an `app.conf` file to it.

In this `app.conf` file we want to have the following text:
```csharp
"app.conf" [readonly] 8L, 268C                                                                                                                                         4,1           All
[program:MembershipApp]
command=/usr/bin/dotnet MembershipApp.WebApi.dll --server.urls:http://*:8888
directory=/home/ec2-user/testing/
autostart=true
autorestart=true
stderr_logfile=/var/log/membership.err.log
stdout_logfile=/var/log/membership.out.log
stopsignal=INT
```

The program name should relate to your app, the command line should be the command to run your app, the directory is where your app is located and the others are self explanatory...

OK - so now we have set up `supervisor` and we have told it about the app, next is to ensure that `supervisor` starts on startup if the Ec2 instance ever goes down and it is instantiated as a `service` within the Linux OS.

To do this we need to execute a few commands - see this link... [install and configure supervisord on centos 7. · GitHub](https://gist.github.com/mozillazg/6cbdcccbf46fe96a4edd)

Now we need to make sure we are the super user for the following commands, so type into the CLI `sudo su`

We have already executed steps 1 and 2, the next step is to setup the service for Linux into the following file `/usr/lib/systemd/system/supervisord.service`, just execute this command:
`wget https://gist.githubusercontent.com/mozillazg/6cbdcccbf46fe96a4edd/raw/2f5c6f5e88fc43e27b974f8a4c19088fc22b1bd5/supervisord.service -O /usr/lib/systemd/system/supervisord.service`

Now we need to edit this file and make sure everything is correct. So lets open the file with `sudo vim /usr/lib/systemd/system/supervisord.service`

The files content should look something like this:

```csharp
[Unit]
Description=supervisord - Supervisor process control system for UNIX
Documentation=http://supervisord.org
After=network.target

[Service]
Type=forking
ExecStart=/bin/supervisord -c /etc/supervisord/supervisord.conf
ExecReload=/bin/supervisorctl reload
ExecStop=/bin/supervisorctl shutdown
User=root

[Install]
WantedBy=multi-user.target
```

One thing I had to change was on the `ExecStart` line the location of the supervisord.conf file needed to be changed to just `/etc/supervisord.conf`

So now to register supervisor as a service with the linux system we need to run launch supervisor and then enable it so run `systemctl start supervisord` & `systemctl enable supervisord`, some other commands are:

start supervisor service:  `systemctl start supervisord`
stop supervisor service: `systemctl start supervisord`
start apache (as a user): `sudo service httpd start`
stop apache( as a user): `sudo service httpd stop`
 
side note: if we have a reverse proxy running (I have apache) we also have to make sure it will run on startup so that our app will work, so to do this we run the command `systemctl enable http.service`



if we want to check the status we can run `systemctl status supervisord`


