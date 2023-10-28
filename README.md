# Jenkins CICD Pipeline with Tomcat (Download - Build - Deploy - Test - Release) 

### Prerequisite:

1) Jenkins Server (One Ec2 Instance)
```
#! /bin/bash -ex
yum update -y
yum install git -y
yum install maven -y
sudo dnf install java-11-amazon-corretto -y
# sudo amazon-linux-extras install java-openjdk11 -y

## For "Amazon Linux 2003" AMI, use command: # sudo dnf install java-11-amazon-corretto -y
## For "Amazon Linux 2" AMI, use : # sudo amazon-linux-extras install java-openjdk11 -y

sudo yum update –y
# Add the Jenkins repo using the following command:
sudo wget -O /etc/yum.repos.d/jenkins.repo \
        https://pkg.jenkins.io/redhat-stable/jenkins.repo
# Import a key file from Jenkins-CI to enable installation from the package:
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade
# Install Jenkins:
sudo yum install jenkins -y
# Enable the Jenkins service to start at boot:
sudo systemctl enable jenkins
# Start Jenkins as a service:
sudo systemctl start jenkins
# You can check the status of the Jenkins service using the command:
sudo systemctl status jenkins

_Add inbound rule to allow traffic on the port 8080_
```

2) Tomcat Server (Two EC2 instances - one for QA environment and another for PROD environment) 
```
#! /bin/bash -ex
yum update -y
yum install git -y
# yum install maven -y
sudo dnf install java-11-amazon-corretto -y
# amazon-linux-extras install java-openjdk11 -y

_Add inbound rule to allow traffic on the port 8080_

# Install Tomcat:
## export VER="9.0.80"
## wget https://archive.apache.org/dist/tomcat/tomcat-9/v${VER}/bin/apache-tomcat-${VER}.tar.gz
sudo wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz
# or https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz
sudo tar -xvf apache-tomcat-9.0.75.tar.gz
# Update permission on Tomcat folder:
sudo chown ec2-user -R /home/ec2-user/apache-tomcat-9.0.75/
# Start Tomcat:
cd /home/ec2-user/apache-tomcat-9.0.75/bin
sh startup.sh
# Allow permission to access Tomcat Manager:
cd /home/ec2-user/apache-tomcat-9.0.75/webapps/manager/META-INF/
vi contex.xml
inside valve section make allow ".*" (remove all other)
# Allow permission to access Tomcat Host-Manager:
cd  /home/ec2-user/apache-tomcat-9.0.75/webapps/host-manager/META-INF
Vi contex.xml
inside valve section make allow ".*" (remove all other)
# Allow permission to users:
cd /home/ec2-user/apache-tomcat-9.0.75/conf/
Vi tomcat-users.xml (add tomct & admin)
<user username="tomcat" password="tomcat" roles="manager-gui"/>
<user username="admin" password="Admin@123" roles="manager-script,manager-status,admin-gui,manager-gui"/>
# Restart Tomcat:
cd /home/ec2-user/apache-tomcat-9.0.75/bin
sh shutdown.sh
sh startup.sh

## Optional - Verify Tomcat From Local PC
git clone https://github.com/ajit-dherange/maven_web_app.git
mvn clean package
check path \\mvn-web-app\target
goto tomcat > manager app > choose war file > deploy 
update index.jsp & war file
goto tomcat > manager app > undeploy
goto tomcat > manager app > choose war file > deploy 
```

### Stage 1: Continuous Download - START CI-CD

	• Add JDK (Manage Jenkins >> Global Tool Configuration > Add JDK > Install Automatically)
  
1) Login to the Jenkins server
2) Create New item as free style project >> Development
4) Click on source code management
5) Select GIT
7) Enter the URL of GitHub repository https://github.com/ajit-dherange/maven_web_app.git
6) Click on apply and save
7) Run the Job (click on Build Now)
8) Check the console output 
9) Connect to the Jenkins server
10) Go to the location where code is downloaded >> $ ls -l workspace_path
	
Workspace_path: /var/lib/jenkins/workspace/Development

### Stage 2 : Continuous Build - Convert the java files in to Artifacts ( .war file)
  
	• Add Maven (Manage Jenkins >> Global Tool Configuration > Maven > Give name: Maven-3.8.6 or 3.9.1)
  
10) Click on configure of the same job Development 
11) Go to Build Section
12) Click on add build step
13) Click on Invoke top level maven targets
14) Enter the goal as  "clean package"
15) click on apply and save
16) Run the Job (click on Build Now)
17) Click on number & click on console output
18) Copy the path of the war file and check the file in the Linux machine >> $ ls -l workspace_path
	
Workspace_path: /var/lib/jenkins/workspace/Development

Artifacts_path: /var/lib/jenkins/workspace/Development/webapp/target/webapp.war

### Stage 3 : Continuous Deployment - Deploy artifact ( .war file) to Container App (Tomcat qa Server)  
	• Install plugin deploy to container (Manage Jenkins >> Manage plugins > Available plugins > Search "deploy to container" > Select and click on install without restart
  
20) Click on post build actions of the Development job
21) Click on add post build actions
22) Click on deploy war/ear to container
23) Enter the path of the war file (or) give "**/*.war" in war/ear files
24) Context path: qaenv
25) Containers: select tomcat9
25) Credentials: Click on add
25) select Jenkins
25) enter tomcat user name and password
25) Click on add
25) Select credentials.
25) give the private ip of the Tomcat qa server >> http://private_ip:8080 > http://172.31.4.56:8080
26) Save the job
27) Run the job (click on Build Now)
28) To access the home page >> http://<public_ip_Tomcat_server>:8080/qaenv >  http://172.31.4.56:8080/qaenv

### Stage 4 Continuous Testing
1) Login to the Jenkins server
2) Create New item as free style project >> Testing
3) Click on source code management 
4) Select GIT
5) Enter the URL of GitHub repository https://github.com/ajit2411/TestingNew
6) Click on apply and save
7) Run the Job (click on Build Now)
8) Check the console output 

### Stage 5  Continuous Delivery
	• Install plugin copy artifacts (Manage Jenkins >> Manage plugins > Available plugins > Search "copy artifacts" > Select and click on install without restart)

1) Go to Development job 
2) Go to Configure
3) Go to Post build actions tab
4) Click on add post build action
5) Click on Archive the artifacts
6) Enter **/*.war
7) Click on apply and save
8) Go to Testing Job
9) Click on configure
10) Go to Build section
11) Click on add build steps
12) Click on copy artifacts from another project
13) Enter Development as project name
14) For Deployment Go to Post build actions section
15) Click on add post build action
16) Click on deploy war/ear to a container
17) Enter **/*.war in war/ear files
18) Context path: prod
19) Click on add container 
20) Select tomcat 9
21) Select your Credentials
22) Enter private ip:8080 of the prod server >> http://172.31.39.130:8080
23) Click on Apply and save
24) Run the job  (click on Build Now)
25) To access the home page >> http://<public_ip_Tomcat_server>:8080/prod >  http://172.31.4.56:8080/prod

### Stage: Email Integration

In case, if a job fails , we need to send notificiation. For that we need to integrate jenkins to smtp server.

We are now integrating jenkins with gmail smtp server. (Search in google "gmail smtp server")

```
Manage Jenkins --- Configure System ----- Email Notification

SMTP Server - smtp.gmail.com
Click on Advance button ( with notepad icon )
USE SMTP Authentication
Username - ajit.dherange@gmail.com
Password - password for the above email
use SSL
SMTP Port - 465

Test Configuration by sending e-mail.
Test email Receipent - sunildevops77@gmail.com

Gmail Settings to get email from jenkins:
1) Goto google account -- Less secure app access --- Allow less secure apps: ON
2) "Disable captcha gmail"
Search in google "Disable captcha gmail" -- Continue
```

ref: https://techviewleo.com/install-tomcat-on-amazon-linux/?expand_article=1
     https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.75/bin/
     
