# Docker-Kubernetes
# PART I: create docker image and push to ECR

1) IAM

create user with AdministratorAccess. Get security credentials of the user

2) EC2 Instance

   Launch ubuntu EC2 instance and install docker and aws cli using user data
	
	#! /bin/bash
	
	sudo apt-get update
	
	sudo apt-get remove docker docker-engine docker.io
	
	sudo apt-get install docker.io -y
	
	sudo systemctl start docker
	
	sudo systemctl enable docker
	
	curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
	
	sudo apt-get update
	
	sudo apt-get install unzip
	
	unzip awscliv2.zip
	
	sudo ./aws/install
  
3) SSH
    	3.1) ssh to EC2 instance and check version of installed tools
    	
	           docker --version
	           aws –version
            
    	3.2) Configure aws credentials
    	
	          aws configure
	          access key
	          secret acces key
  
	3.3) To enter docker command without “sudo”, add user to docker group
	
	          sudo usermod -aG docker $USER
	     Re-login to the ssh
  
4) Static content

      4.1) create index.html file
      
	         vi index.html
		    
      4.2) paste content from index.html file
      
5) Dockerfile

	FROM nginx
	
	ADD index.html /usr/share/nginx/html

   NOTE: Default document root for Apache: /var/www/html

   NOTE: Default document root for Nginx: /usr/share/nginx/html

6) Docker image

     docker build -t imageName . 

7) Docker Container

    docker run -d -p 80:80 imageName

8) View your application

     paste public IP of ubuntu in browser
      
9) ECR Repo

   Create public repo in AWS ECR

10) Push image to ECR

	Get push command from AWS ECR
	
	10.1) Authenticate

	aws ecr get-login-password --region <yourRegion> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<yourRegion>.amazonaws.com
	
	10.2) Tag image
	
	docker tag <yourimagename:tag> <yourRepoURL/repoName:tag>
	
	10.3) Push image to AWS ECR
	
	docker push aws_account_id.dkr.ecr.us-west-2.amazonaws.com/repoName:tag

11) Deploy
	
    Use this image in ECR to deploy in EKS in part II 
	
# PART II: Deploy application on kubernetes

