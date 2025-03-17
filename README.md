# Project-2-Tomcat-Integration
## Integrating Tomcat server in CI/CD Pipeline

### Step1: Setup Tomcat Server

- First create an EC2 instance for Tomcat Server
```sh
instanceType: t2.micro
AMI: Amazon Linux-2
Security Group: 
22, SSH
8080, Custom TCP
```

- Next install java-11 in Tomcat server, switch to root user `sudo su -` and run below command. Once installed, check `java -version`

```sh
amazon-linux-extras install java-openjdk11
```

- Next we will install Tomcat, switch to `/opt` directory 

```sh
cd /opt
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.33/bin/apache-tomcat-10.1.33.tar.gz
tar -xvf apache-tomcat-10.1.33.tar.gz
mv apache-tomcat-10.1.33 tomcat
sudo rm -r apache-tomcat-10.1.33.tar.gz

```
- Now we can start our Tomcat server

```sh
cd tomcat/bin/
./startup.sh
```
- Now we should be able to access our Tomcat server from browser. Go to `http:<public_ip_of_your_tomcat_server>:8080`

- When we click `Manager App` we should get `403 Access Denied` error. To fix this issue we need to edit `context.xml` file.

```sh
By default the Manager is only accessible from a browser running on the same machine as Tomcat. If you wish to modify this restriction, you'll need to edit the Manager's context.xml file.
``` 

- Go to tomcat directory run `find / -name context.xml` to get the location of `context.xml` file

```sh
/opt/tomcat/webapps/examples/META-INF/context.xml
/opt/tomcat/webapps/host-manager/META-INF/context.xml
/opt/tomcat/webapps/manager/META-INF/context.xml
```

- We should update `context.xml` file under `host-manager` and `manager` directories. Currently it is only allowing access from localhost, we will comment out the part shown below in the xml.

```sh
<!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
  allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
```

- Once we updated those files, we need to stop and restart our Tomcat server. 
```sh
cd tomcat/bin/
./shutdown.sh
./startup.sh
```

- Now we updated the files, we will no longer get `403 Access Denied` error but it will ask username&password in Tomcat server. To find the credentials, go to under tomcat/conf directory
```sh
cd tomcat/conf/
vim tomcat-users.xml
```

- We will add below users  to the file, and save the file.

```sh
 <role rolename="manager-gui"/>
 <role rolename="manager-script"/>
 <role rolename="manager-jmx"/>
 <role rolename="manager-status"/>
 <user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
 <user username="deployer" password="deployer" roles="manager-script"/>
 <user username="tomcat" password="s3cret" roles="manager-gui"/>
``` 

- Once we updated the file, we need to stop and restart our Tomcat server. 
```sh
cd tomcat/bin/
./shutdown.sh
./startup.sh
```

### Step2: Integrate Tomcat with Jenkins
