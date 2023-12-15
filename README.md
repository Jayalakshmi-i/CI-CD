# Automated deployment of a WAR file to Tomcat Server through CI/CD Pipeline


This project focuses on implementing a robust Continuous Integration and Continuous Deployment (CI/CD) pipeline using tools like Git, Jenkins, Dockerfile, and Maven within the AWS Cloud environment. Configured Jenkins jobs to automatically trigger builds upon new code commits, build efficiently and deploy WAR files to a Tomcat server on EC2 instance, and consistently delivery to web applications.

                              
![CI_CD drawio](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/6ac088da-e782-4e0f-a6f8-3ba46420c49b)

### Step 1: Launched an Ec2 instance with Amazon Linux image and t2.micro instance type and installed Jenkins and Java, git using user-data script
```
#!/bin/bash

sudo yum update -y

sudo wget -O /etc/yum.repos.d/jenkins.repo \https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

sudo yum upgrade -y
sudo amazon-linux-extras install java-openjdk11 -y

sudo yum install jenkins -y
sudo systemctl daemon-reload
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo yum install git -y
hostname Master-server
```
### Step 2: Next, installed Maven using the following commands:
```
cd /opt/
wget https://dlcdn.apache.org/maven/maven-3/3.9.5/binaries/apache-maven-3.9.5-bin.tar.gz     //To download Maven tar.gz file
tar -xvzf apache-maven-3.9.5-bin.tar.gz             //Unzipping downloaded tar.file
mv apache-maven-3.9.5 maven
cd maven/bin
./mvn -v
```
### Step 3: Edited .bash_profile file to update Maven and Java installation path:
```
// to change user to root
sudo su - 
cd ~
//edit .bash_profile file 
vim .bash_profile
//Add the following data under a user-specific environment
M2=/opt/maven/bin
M2_HOME=/opt/maven
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.20.0.8-1.amzn2.0.1.x86_64/bin/java
PATH=$PATH:$HOME/bin: $M2:$M2_HOME:$JAVA_HOME
```

### Step 4: Setting up Java and Maven in Jenkins
Setting up the JAVA_HOME and MAVEN_HOME environment variable in our Jenkins system

<img align=center img src="https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/ced7667b-c54e-4005-87ae-a10244dd24f7" alt="Jenkins" class="center" style="margin-bottom: 50px 50px;">

<br><img src="https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/a5570f68-6106-4ee4-8cec-8cfe37d88063" alt="Java" class="center">

<img src="https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/39ba260e-a0fc-4b97-bf8c-fe2e3a611147" alt="Jenkins" class="center"> 

### Step 5: Installed Docker on the Docker server, pulled the tomcat image from Docker hub, and created Ducker container using it on port 8081

Tomcat container started on port 8081: http://18.223.235.177:8081 with “HTTPS status 404 – not found” error message as there are no files on <b>/usr/local/tomcat/webapp</b> directory where Tomcat default files will load when we access.

### Step 6: Created Docker group user credentials to connect with our Docker server. For this created a new user in the Docker server

On the Docker server, editing /etc/ssh/sshd_confing file and uncommented
<b>“PasswordAuthentication yes”</b> line and commented <b>PasswordAuthentication no</b> line to provide root privileges for the Docker server.

Restarted the sshd service for the new changes to happen using the below command:

```sudo service sshd reload```

Adding this user to the docker user group. Get the docker service username from the <b>/etc/group</b> file and execute the below command to add the <b>“dockeradmin”</b> to the docker service group.

```sudo usermod -aG docker dockeradmin```

### Step 7: Integrating the Docker server with Jenkins using Publish over SSH plugin:

### Step 8: Creating Dockerfile

### Running Dockerfile

Created Tomcat container from Docker image which is pulled from Dockerhub using Dockerfile:

```sudo docker run -d --name tomcat-app -p 8088:8080 new-tomcat```

Created the new container named tomcat-app and accessed the public IP address of the docker host on the browser to confirm the application is loading and it loads fine at: http://18.223.235.177:8088 

### Step 9: Created a new job with the Freestyle project and made the following changes:
 
 
 
Uncommented the copying files line in Dockerfile:
 

Post-build actions: 
![17](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/8ebdde11-335b-4d3a-9d0b-0c5482fe0ee0)

![re1](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/41859eef-48d4-4342-a23d-ed7754b32732)

![re2](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/109b1893-206d-4041-af22-270b6b679d20)


### Deployed war file at Tomcat server: http://18.223.235.177:808/webapp/
![result](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/24e53a5b-ada7-4449-a5b0-8cb9cde7e04a)


