## Install and configure sonarqube
## PREREQUISITES
### We will make some Linux Kernel configuration changes to ensure optimal performance of the tool – we will increase vm.max_map_count, file discriptor and ulimit.
```
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```
### To make a permanent change, edit the file /etc/security/limits.conf and append the below
```
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
```

# STEPS
```
1- change password for default user postgres: password = 8989
2- switch to default user postgres `su - postgres` and create a new user sonar
3- create password for user "sonar" 
- switch to postgreSQL shell $psql
- set user "sonar" password to be "sonar" $ALTER USER sonar WITH ENCRYPTED password 'sonar';
4- create database sonarqube with owner set as the user sonar. $CREATE DATABASE sonarqube OWNER sonar;
5- grant all privileges on database sonarqube to user sonar $grant all privileges on DATABASE sonarqube to sonar;
6- exit shell and switch out of user postgres `\q` `exit`
```

![step1-8](./sonar%20installation%20images/step1-8.jpg)

# step 9- Then install sonarqube:
### Navigate to the tmp directory to temporarily download the installation files
 `cd /tmp && sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.3.zip`

### step 10- Unzip the archive setup to /opt directory
`sudo unzip sonarqube-7.9.3.zip -d /opt`
### step 11- Move extracted setup to /opt/sonarqube directory
`sudo mv /opt/sonarqube-7.9.3 /opt/sonarqube`

### you cannot run SonarQube as a root user, if you run using root user it will stop automatically. The ideal approach will be to create a separate group and a user to run SonarQube

### Remember on postgres you created a database sonarqube, and you created a user sonar with all privileges to run the database.

### Now that you have installed sonarQube, you need to create a separate group and user to run sonarQube

### Create a group sonar
`sudo groupadd sonar`

### Add a user named sonar with it's home dir set to /opt/sonarqube and added to the group sonar. Also give user control over /opt/sonarqube dir
`sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar`
`sudo chown sonar:sonar /opt/sonarqube -R`

### Open SonarQube configuration file using your favourite text editor, Find the following lines: it is usually in the PostgreSQL section

```
#sonar.jdbc.username=
#sonar.jdbc.password=
```
### Uncomment them and provide the values of PostgreSQL Database username and password:

```
sudo vim /opt/sonarqube/conf/sonar.properties
sonar.jdbc.username=sonar
sonar.jdbc.password=8989
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```

### Edit the sonar script file and set RUN_AS_USER
`Edit the sonar script file and set RUN_AS_USER`
`RUN_AS_USER=sonar`
### Now to start sonarqube

switch to `sonar` user, move to script directory and run the script, check ths status to confirm it is running
```
sudo su sonar
cd /opt/sonarqube/bin/linux-x86-64/
./sonar.sh start
./sonar.sh status
tail /opt/sonarqube/logs/sonar.log
```
### Configure it to run as systemd service

### Create a systemd service file for SonarQube to run as System Startup.

` sudo nano /etc/systemd/system/sonar.service`

### Add the follwing configuration below
```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

### Save the file and control the service with systemctl
```
sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar
```

### Access sonarqube
`http://server_IP:9000/sonar OR http://localhost:9000`

c1caa9233f7dd62a9873f97c06dc9807147e9fed