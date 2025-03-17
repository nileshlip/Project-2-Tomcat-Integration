# Project-2-Tomcat-Integration
ntegrating Tomcat server in CI/CD Pipeline
Step1: Setup Tomcat Server
First create an EC2 instance for Tomcat Server
instanceType: t2.micro
AMI: Amazon Linux-2
Security Group: 
22, SSH
8080, Custom TCP
Next install java-11 in Tomcat server, switch to root user sudo su - and run below command. Once installed, check java -version
amazon-linux-extras install java-openjdk11
Next we will install Tomcat, switch to /opt directory
cd /opt
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.33/bin/apache-tomcat-10.1.33.tar.gz
tar -xvf apache-tomcat-10.1.33.tar.gz
mv apache-tomcat-10.1.33 tomcat
sudo rm -r apache-tomcat-10.1.33.tar.gz
Now we can start our Tomcat server
cd tomcat/bin/
./startup.sh
Now we should be able to access our Tomcat server from browser. Go to http:<public_ip_of_your_tomcat_server>:8080

When we click Manager App we should get 403 Access Denied error. To fix this issue we need to edit context.xml file.

By default the Manager is only accessible from a browser running on the same machine as Tomcat. If you wish to modify this restriction, you'll need to edit the Manager's context.xml file.
Go to tomcat directory run find / -name context.xml to get the location of context.xml file
/opt/tomcat/webapps/examples/META-INF/context.xml
/opt/tomcat/webapps/host-manager/META-INF/context.xml
/opt/tomcat/webapps/manager/META-INF/context.xml
We should update context.xml file under host-manager and manager directories. Currently it is only allowing access from localhost, we will comment out the part shown below in the xml.
<!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
  allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
Once we updated those files, we need to stop and restart our Tomcat server.
cd tomcat/bin/
./shutdown.sh
./startup.sh
Now we updated the files, we will no longer get 403 Access Denied error but it will ask username&password in Tomcat server. To find the credentials, go to under tomcat/conf directory
cd tomcat/conf/
vim tomcat-users.xml
We will add below users to the file, and save the file.
 <role rolename="manager-gui"/>
 <role rolename="manager-script"/>
 <role rolename="manager-jmx"/>
 <role rolename="manager-status"/>
 <user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
 <user username="deployer" password="deployer" roles="manager-script"/>
 <user username="tomcat" password="s3cret" roles="manager-gui"/>
Once we updated the file, we need to stop and restart our Tomcat server.
cd tomcat/bin/
./shutdown.sh
./startup.sh
Step2: Integrate Tomcat with Jenkins
Install Deploy to Container plugin in Jenkins. Go to Manage Jenkins -> Manage Plugins, find the plugin under Available and choose install without restart

Configure Tomcat Server with credentials. Go to Manage Jenkins -> Manage Credentials. We will select Add credentials. We will use the credential we have added to tomcat-user.xml file for this step. Since these credentials will be used for deploying app, we will add deployer credentials which has manager-script role.

Kind: username with password
username: deployer
pwd: deployer
Now we can create our next job with name of BuildAndDeployJob. After build step, the artifact will stored under webapp/target/ directory as webapp.war.
Kind: Maven Project
SCM: https://github.com/nileshlip/hello-world-Projects.git
Goal and options: clean install
Post Build Actions: Deploy war/ear to a container
WAR/EAR files: **/*.war
Container: Tomcat 8 (even though we have install v9, this plugin is giving some issues with v9, for that reason we will use v8)
Credentials: tomcat_deployer
Tomcat URL: http://<Public_IP_of_Tomcat_server>:8080/ 
Save and Build the job. When go to Tomcat server under Manager App, you will be able to see our application under webapp/
