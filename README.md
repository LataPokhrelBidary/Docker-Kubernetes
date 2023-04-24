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

1)	Create IAM user 
AWS management console ------>IAM----> create user---->policy: AdministratorAccess
Get security credentials

2)	Launch/configure EC2 instance (client machine) to manage K8S cluster.
a.	Use default vpc
b.	Use user data for instance to install AWS cli, eksctl, kubectl.

#! /bin/bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt-get update
sudo apt-get install unzip
unzip awscliv2.zip
sudo ./aws/install
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin

c.	ssh to client machine 
d.	check for installed tool
i.	$ aws –version
ii.	$ eksctl version
iii.	$ kubectl version –short –client
e.	Configure client machine with credentials
i.	$ aws configure
ii.	$ paste access key from step 1
iii.	$ paste secret access key from step 1
iv.	$ choose your aws region
3)	Create EKS cluster using eksctl command

$ eksctl create cluster --name=mycluster \
               	--region=us-east-1 \
                         	--zones=us-east-1a,us-east-1b \
                      	--nodegroup-name mynodegroup \
                      	--node-type=t3.medium \
                      	--nodes=2 \
                      	--nodes-min=2 \
                      	--nodes-max=4 \
                      	--managed

4)	Resource created: above command will create following resources.
•	VPC and public and private subnets
•	EC2 & Autoscaling group
•	CloudFormation stack
•	EKS cluster and node group
The cluster will have 2 worker nodes of t3.medium EC2 instance and will be placed in public subnet by default.
5)	Kubeconfig: Above command also update kubeconfig file in client machine so client machine can connect to the cluster. Kubectl looks for a file named config in the $HOME/.kube directory. kubeconfig contains information about clusters, users, namespaces, and authentication mechanisms.
 
6)	Create deployment/service using kubectl
a.	vi deploy.yaml
b.	press i to go to insert mode
c.	paste content from deploy.yaml file
d.	esc :wq
e.	$ kubectl apply -f deploy.yaml
7)	Above command will create deployment, replica set, pod, service of type load balancer
a.	$ kubectl get deploy
b.	$ kubectl get rs
c.	$ kubectl get po
d.	$ kubectl get svc
e.	Get load balancer DNS name and paste in browser to view your application
8)	Delete a cluster: You must delete the service first
a.	Manual way: going to console and delete resources
b.	Using eksctl command
i.	$ eksctl delete cluster --name mycluster --region us-east-1

