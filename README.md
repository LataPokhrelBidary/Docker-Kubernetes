# Docker-Kubernetes
# PART I: create docker image and push to ECR
![docker demo part I](https://user-images.githubusercontent.com/72663705/234431822-856e5a44-bf59-4c99-b586-722eaf90130c.jpg)

1) IAM

   create user with AdministratorAccess. Get security credentials of the user

2) EC2 Instance

   Launch ubuntu EC2 instance and install docker and aws cli using userdata1.txt
		  
3) SSH

    	3.1) connect (ssh) to EC2 instance and check version of installed tools
    	
	           docker --version
	           aws –version
            
    	3.2) use IAM user's credentials to confiugre instance
    	
	          aws configure
	          access key
	          secret acces key
  
	3.3) To enter docker command without “sudo”, add user to docker group and Re-login to the ssh
	
	          sudo usermod -aG docker $USER
			       
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
	
![k8spartII](https://user-images.githubusercontent.com/72663705/234432282-f08b8f33-9bd9-4fc3-9521-71f8db382daa.jpg)

1) Create IAM user 
   AWS management console ------>IAM----> create user---->policy: AdministratorAccess
   Get security credentials

2) Launch/configure EC2 instance (client machine) to manage K8S cluster.
	
		a. Use default vpc
		b. Use userdata2.txt for instance to install AWS cli, eksctl, kubectl.
		c. ssh to client machine 
		d. check for installed tool
	
			aws –version
			eksctl version
			kubectl version –short –client
	
		e. Configure client machine with credentials
	
			aws configure
			paste access key from step 1
			paste secret access key from step 1
			choose your aws region
	
3) Create EKS cluster using eksctl command

		eksctl create cluster --name=mycluster \
               		--region=us-east-1 \
                        --zones=us-east-1a,us-east-1b \
                      	--nodegroup-name mynodegroup \
                      	--node-type=t3.medium \
                      	--nodes=2 \
                      	--nodes-min=2 \
                      	--nodes-max=4 \
                      	--managed

4) Resource created: above command will create following resources.
	
		• VPC and public and private subnets
	
		• EC2 & Autoscaling group
	
		• CloudFormation stack
	
		• EKS cluster and node group
	
   The cluster will have 2 worker nodes of t3.medium EC2 instance and will be placed in public subnet by default.
	
5) Kubeconfig: Above command also update kubeconfig file in client machine so client machine can connect to the cluster. Kubectl looks for a file named config in the 	$HOME/.kube directory. kubeconfig contains information about clusters, users, namespaces, and authentication mechanisms.
 
6) Create deployment/service using kubectl
	
		a. vi deploy.yaml
	
		b. press i to go to insert mode
	
		c. paste content from deploy.yaml file
	
		d. esc :wq
	
		e. $ kubectl apply -f deploy.yaml
	
7) Above command will create deployment, replica set, pod, service of type load balancer
	
		kubectl get deploy
	
		kubectl get rs
	
		kubectl get po
	
		kubectl get svc
	
   Get load balancer DNS name and paste in browser to view your application
	
8) Delete a cluster: You must delete the service first
	
		a. Manual way: going to console and delete resources
	
		b. Using eksctl command
	
			 eksctl delete cluster --name mycluster --region us-east-1
