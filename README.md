# Jenkins CICD Pipeline with Tomcat

### Prerequisite:

1) Jenkins Server
```
#! /bin/bash -ex
yum update -y
yum install git -y
# sudo dnf install java-11-amazon-corretto -y
sudo amazon-linux-extras install java-openjdk11 -y

For "Amazon Linux 2003" AMI, use command: # sudo dnf install java-11-amazon-corretto -y
For "Amazon Linux 2" AMI, use : # sudo amazon-linux-extras install java-openjdk11 -y

Add inbound rule to allow traffic on the port 8080
```

2) Tomcat Server
```
#! /bin/bash -ex
sudo yum update -y
# sudo dnf install java-11-amazon-corretto -y
sudo amazon-linux-extras install java-openjdk11 -y

java --version

sudo wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz
sudo tar -xvf apache-tomcat-9.0.75.tar.gz

sudo chown ec2-user -R /home/ec2-user/apache-tomcat-9.0.75/
sh startup.sh

Add inbound rule to allow traffic on the port 8080

Vi /home/ec2-user/apache-tomcat-9.0.75/webapps/manager/META-INF/contex.xml
inside valve section make allow ".*" (remove all other)

Vi /home/ec2-user/apache-tomcat-9.0.75/webapps/host-manager/META-INF/contex.xml
inside valve section make allow ".*" (remove all other)

Vi /home/ec2-user/apache-tomcat-9.0.75/conf/tomcat-users.xml (add tomct & admin)
<user username="tomcat" password="tomcat" roles="manager-gui"/>
<user username="admin" password="Admin@123" roles="manager-script,manager-status,admin-gui,manager-gui"/>

git clone https://github.com/MRaju2022/mvn-web-app.git
mvn clean package
check \\mvn-web-app\target
goto tomcat > manager app > choose war file > deploy 
update index.jsp & war file
goto tomcat > undeploy
goto tomcat > choose war file > deploy 
```

### Stage 1: Continuous Download - START CI-CD

	• Add JDK (Manage Jenkins >> Global Tool Configuration > Add JDK > Install Automatically)
  
1) Login to the Jenkins server
2) Create New item as free style project
4) Click on source code management
5) Select GIT
7) Enter the URL of GitHub repository https://github.com/MRaju2022/maven.git
https://github.com/MRaju2022/mvn-web-app.git
https://github.com/ajit2411/maven.git
https://github.com/sunildevops77/maven.git
https://github.com/jleetutorial/maven-project.git
6) Click on apply and save
7) Run the Job (click on Build Now)
8) Check the console output 
8) Connect to the Jenkins server
9) Go to the location where code is downloaded
Ls -l <path> (/var/lib/jenkins/workspace/demojob)

### Stage 2 : Continuous Build - Convert the java files in to artifact ( .war file)
  
	• Add Maven (Manage Jenkins >> Global Tool Configuration > Maven > Give name: Maven-3.8.6
  
10) Click on configure of the same job
  
11) Go to Build Section
  
12) Click on add build step
  
13) Click on Invoke top level maven targets
  
14) Enter the goal as  "clean package"
  
15) click on apply and save
  
16) Run the Job
  
17) Click on number & click on console output
  
18) Copy the path of the war file and check the file in the Linux machine
Ls -l <path> (/var/lib/jenkins/workspace/demojob/webapp/target/webapp.war)

workspace /var/lib/jenkins/workspace/DevOps01
/var/lib/jenkins/workspace/DevOps01/webapp/target/webapp.war

### Stage 3 : Continuous Deployment - Deploy artifact ( .war file) to Container App
  
Now we need to deploy the war file into the Tomcat (Test) Server.
  
19) install "deploy to container" plugin.
Go to Dashboard > Click on manage Jenkins > Click on manage plugins > Click on available section > Search for plugin ( deploy to container)
Select that plugin and click on install without restart.
  
20) Click on post build actions of the development job
  
21) Click on add post build actions
  
22) Click on deploy war/ear to container
  
23) Enter the path of the war file (or)
 we can give **/*.war in war/ear files.
  
24) Context path: maven
  
25) Containers : select tomcat 9
  
Credentials : Click on add
  
select Jenkins
  
enter tomcat user name and password
  
Click on add
  
Select credentials.
  
give the private ip of the Tomcat server (Test).
  
http://private_ip:8080 >> http://172.31.4.56:8080
  
27) Run the job
  
28) To access the home page
  
http://<public_ip_Tomcat_server>:8080/maven  >>  http://172.31.4.56:8080/maven
  

