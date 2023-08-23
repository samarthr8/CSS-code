DevOps CI/CD Flow

Description - DevOpsCI/CDFlow is a comprehensive Continuous Integration and Continuous Deployment (CI/CD) pipeline that automates the process of building, testing, and deploying frontend code using Jenkins, SonarQube, and Docker. The project is designed to streamline the development workflow and ensure code quality and consistency throughout the deployment process.

Features:
Jenkins Integration: Jenkins, a powerful automation server, is used to orchestrate the CI/CD pipeline. It automatically triggers builds upon new code commits and handles the entire deployment process.
SonarQube Code Analysis: The code pushed to the GitHub repository is subjected to thorough code analysis using SonarQube. This step ensures code quality, identifies bugs, and enforces best coding practices.
Docker Containerization: The frontend code is seamlessly deployed using Docker. Docker containers enable consistent deployment across various environments, simplifying the deployment process and minimizing conflicts.
Multiple EC2 Machines: The project utilizes three EC2 machines, each dedicated to hosting Jenkins, SonarQube, and Docker, respectively. This segregation ensures better resource management and security.
Usage:
To utilize this CI/CD pipeline, follow these steps:
Clone this repository to your local machine and make necessary configuration changes.
Push your frontend code to the designated GitHub repository.
Jenkins will automatically detect the new commit and trigger the CI/CD pipeline.
The code will be pulled from the GitHub repository and sent for code analysis to SonarQube.
After passing the code analysis, the code will be deployed using Docker.
The Project flow represented on a Diagram:








Steps: 
Download the free css code from any website and push that code to a new github repository.
Launch 3 ec2 machines namely - jenkins, sonarqube and docker respectively.
SSH into the jenkins instance and install openjre11 and jenkins:
To update packages:	sudo apt update
To install Java RE 11:	sudo apt install openjdk-11-jre -y
To install jenkins: copy and paste the whole command
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins 
Open port 8080 in the jenkins machine security group for my IP. Setup jenkins using <public ip:8080> 
Create a freestyle project with any name say <Automated Pipeline>. 
Paste the github repo details in the Source Code Management section.
Check <GitHub hook trigger for GITScm polling> to trigger pipeline built after every change in the github repo.
Add a github WebHook:
Go to setting in your github repository.
Select webhooks and create a new one.
Type <http://44.211.247.134:8080/github-webhook/> into payload url tab.
Select: Let me select individually > Pull requests, Pushes and Save
Make changes to the github repository and verify if jenkins job is triggered.


SSH into the SonarQube machine. 
Change its hostname using command:
sudo hostnamectl set-hostname SonarQube
/bin/bash
Similarly change the hostname of Jenkins machine aswell.
 Install SonarQube on that machine:
Update packages:		sudo apt update
Install Java 17:		sudo apt install openjdk-17-jre
Install Sonarqube zip file:
	wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.1.0.73491.zip
Install Unzip Package:	sudo apt install unzip
Unzip package:		unzip sonarqube-10.1.0.73491.zip


To configure Sonarqube run the following commands:
	cd sonarqube-10.1.0.73491/bin/linux-x86-64/
	./sonar.sh console
	Open 9000 port for myIP in security group
	Verify if the output says ‘SonarQube is operational’
	Visit the SonarQube page by <public ip:9000> . 
	Default username and password is admin admin.
	Set new password ‘samarth’
 Select Manually on Sonarqube console. Give the following details.
 Project display name: Sam-website-scan
 Project key: Sam-website-scan
 Branch: main
 Next
Define a specific setting for this project > Previous Version > Create Project> Select Jenkins> Select  Github> Configure Analysis > Continue > Continue > Other > Copy Project Key to a notepad [sonar.projectKey=Samarth-website-scan] > Finish the  tutorial >
To create a token: 
Go to Admin user (top right corner) > My account > Security > Generate token:
Name : SonarQube-Token
Type: Global analysis token
Validity: 30 day 
Generate token and copy the token to notepad [sqa_a16cb8da529e592b0f48a71d99b0685651a22e3d]


Go to jenkins machine and install plugins SonarQube Scanner and SSH2 Easy
Now we need to do Sonarqube Scanner configure it.
Go to manage jenkins > Tools > SonarQube Scanner > Name : SonarScanner(Choose your own) > install automatically > Save
Go to configure Systems > Sonarqube Servers > Add server > Name : Sonar-server > Server Url : http://174.129.191.177:9000 > Save 
Go to your pipeline > Configure > Build steps > Execute SonarQube Scanner > Paste the project key from notepad in ‘Analysis properties’
Go to Manage jenkins > configure Systems > Sonarqube Servers > Add token > Jenkins > Secret text > add secret text  > id: Sonar-token > select this token.
Run the Jenkins job to see if the output is Success.


SSH into the Docker machine and install docker:
	sudo apt update
	sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
	echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	sudo apt update
	apt-cache policy docker-ce
	sudo apt install docker-ce -y
	sudo usermod -aG docker ${USER}
 Go to jenkins server 
	switch to jenkins user:  sudo su jenkins 
	try to ssh into docker:   ssh ubuntu@dockerIP
	Set password for ubuntu user:    passwd ubuntu
	Password: admin
Go to docker server run 
	 sudo su 
	 vim /etc/ssh/sshd_config
	Uncomment line < PubkeyAuthentication yes >
	Change < PasswordAuthentication yes >
	systemctl restart sshd
Go to Jenkins server:
	ssh ubuntu@dockerIP  : to get success we need to change 22 port access to Anywhere in SG
	Give the password set above when prompted. Now we are into docker server through Jenkins.
	Exit
	ssh-keygen 		Press enter three times
	ssh-copy-id ubuntu@dockerIP
	Now we can go inside docker server without mentioning password
	
 Go to Jenkins console > Manage Jenkins > System > Server Groups Center > Add Server Group List > 
 Group Name : Docker-Servers
 SSH port : 22
 Username : ubuntu 
 Password : admin (as set above)
 Save


Go to Jenkins console > Manage Jenkins > System > Server Groups Center > Add Server List > 
 Server Group : Docker-Servers
 Server Name : Docker-1
 Server IP : 34.236.158.162 
 Save


Go to Pipeline > Configure > Add Build Step > Remote Shell > touch test.txt > save > Build Pipeline
Verify is the test.txt file is created on docker machine or not


Go to github repo and create a file Dockerfile: 
FROM nginx
COPY . /usr/share/nginx/html/
Save the file. Pipeline would be triggered.


 Go to pipeline > Configure > Remove Remote shell step > Add Execute Shell 
 scp -r ./* ubuntu@34.236.158.162:~/website/
 Before that create a folder name <website > in docker server
 We see Sonarqube scan failed because we put a dockerfile in the repository which is a bad practice.   But let's ignore it for now.
Save and Build the pipeline
 If got success. To verify go to docker server go to website folder to see the github repo content there.


 Add a remote shell command to pipeline:
cd /home/ubuntu/website/
docker build -t mywebsite .
docker run -d -p 8085:80 --name=Samarth-website mywebsite
Save


 Now Build the pipeline.
 Open the port 8085 for everyone.
 Go to docker server < publicip:8085 > to view the website


 To deploy again you have to go to docker server and delete the container and image. Then the pipeline can be built successfully. 
Successfully Deployed!!!!!
