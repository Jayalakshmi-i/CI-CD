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
Launched Jenkins application at http://3.142.148.106:8080
<img src="https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/ced7667b-c54e-4005-87ae-a10244dd24f7" alt="Jenkins" class="center">
### Step 2: Next, install Maven using the following commands:
```
cd /opt/
wget https://dlcdn.apache.org/maven/maven-3/3.9.5/binaries/apache-maven-3.9.5-bin.tar.gz     //To download Maven tar.gz file
tar -xvzf apache-maven-3.9.5-bin.tar.gz             //Unzipping downloaded tar.file
mv apache-maven-3.9.5 maven
cd maven/bin
./mvn -v
```
### Step 3: Editing .bash_profile file to update Maven and Java installation path so we can access them outside the "bin" folder/path :
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
### Accessing the maven outside the /bin folder
![13](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/4cd22ec3-4b89-4b6b-a8a0-8dbf22f64e88)

### Step 4: Setting up Java and Maven in Jenkins
Setting up the JAVA_HOME and MAVEN_HOME environment variable in our Jenkins system

<br><img src="https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/a5570f68-6106-4ee4-8cec-8cfe37d88063" alt="Java" class="center">

<img src="https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/39ba260e-a0fc-4b97-bf8c-fe2e3a611147" alt="Maven" class="center"> 

### Step 5: Luanched another Ec2 instance for the Docker server to deploy a war file and installed Docker in it.
Pulled the tomcat image from the Docker hub, and created a Docker container using it on port 8081

Tomcat container started on port 8081: http://18.223.235.177:8081 with “HTTPS status 404 – not found” error message as there are no files on <b>/usr/local/tomcat/webapp</b> directory where Tomcat default files will load when we access.

![3](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/6d032ad9-a346-496f-a63a-4c7a851978d0)
![4](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/47ec4bd2-6131-4554-9c20-2e46482a5c95)

### Step 6: Created Docker group user credentials to connect with our Docker server through Jenkins. For this, I created a new user in the Docker server:
```
useradd dockeradmin   //"dockeradmin" is the name of the docker user
passwd dockeradmin    //Password for the "dockeradmin" user
```

On the Docker server, editing <b>/etc/ssh/sshd_confing</b> file and uncommented <b>“PasswordAuthentication yes”</b> line and commented <b>PasswordAuthentication no</b> line to provide root privileges for the Docker server.

![5](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/fa534cfd-3730-40da-ac46-b2dd6355d388)

Restarted the sshd service for the new changes to happen using the below command:

```sudo service sshd reload```

Adding this user to the docker user group. Get the docker service username from the <b>/etc/group</b> file and execute the below command to add the <b>“dockeradmin”</b> to the docker service group.

```sudo usermod -aG docker dockeradmin```

### Step 7: Integrating the Docker server with Jenkins using Publish over SSH plugin:
![7](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/fb1a0a8f-b3ac-4158-9265-6cf2edba91a0)

### Step 8: Created Dockerfile to automate a few tasks

### Running Dockerfile
![Capture11](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/0693c07c-2045-4964-a3bc-786fa6bd5985)

Created Tomcat container from Docker image which is pulled from Dockerhub using Dockerfile:

```sudo docker run -d --name tomcat-app -p 8088:8080 new-tomcat```

Created the new container and Tomcat application is installed on the Docker server using the Dockerfile and it loads fine at: http://18.223.235.177:8088 
![tom](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/d617b064-78aa-40c8-a9bb-9106c2831cee)

### Step 9: Created a new job with the Freestyle project and made the following changes:

![8](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/86c86926-4dab-4575-bef6-5c2bfbae9df9)
![9](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/3a73d528-86bd-46ed-aaf2-86d7c9f91a93)
![15](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/c3271bcd-0dc9-454a-9237-7b0febafbbda)

![16](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/4716574a-8605-4c9a-ad7d-d279cac4a7de)

### Uncommented the last line in Dockerfile:
![Capture12](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/bb761a1c-8bfa-48c8-9908-f602af44d23b)

### Step 10: Configuring Post-build actions to deploy a updated war file: 
![17](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/8ebdde11-335b-4d3a-9d0b-0c5482fe0ee0)

![re1](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/41859eef-48d4-4342-a23d-ed7754b32732)

![re2](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/109b1893-206d-4041-af22-270b6b679d20)

![19](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/cdc304bd-282c-40de-9f83-a05057655b60)

### Deployed war file at Tomcat server: http://18.223.235.177:8088/webapp/
![result](https://github.com/Jayalakshmi-i/CI-CD/assets/141424247/24e53a5b-ada7-4449-a5b0-8cb9cde7e04a)


