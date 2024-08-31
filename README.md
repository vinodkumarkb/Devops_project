
# DevOps Project Outline

1) Setting up Terraform to facilitate infrastructure provisioning.

2) Using Terraform to provision Jenkins master, build nodes, and Ansible.

3) Configuring an Ansible server.

4) Employing Ansible to configure Jenkins master and build nodes.

5) Creating a Jenkins pipeline job.

6) Developing a Jenkinsfile from scratch.

7) Configuring SonarQube and Sonar scanner.

8) Executing SonarQube analysis for code quality assessment.

9) Configuring JFrog Artifactory.

10) Creating Dockerfile for containerization.

11) Storing Docker images on Artifactory.

12) Utilizing Terraform to provision a Kubernetes cluster.

13) Creating Kubernetes objects.

14) Deploying Kubernetes objects using Helm.

15) Setting up Prometheus and Grafana using Helm charts.

16) Monitoring the Kubernetes cluster using Prometheus.

# Step-1 : Prepare Terraform Environment on Windows

As part of this, we should setup

Terraform
VS Code
AWSCLI

1) Download terraform the latest version from official website

2) Setup environment variable click on start --> search "edit the environment variables" and click on itUnder the advanced tab, chose "Environment variables" --> under the system variables select path variable and add terraform location in the path variable. system variables --> select path
add new --> terraform_Path

in my system, this path location is C:\terraform_1.3.7

3) Run the below command to validate terraform version

	$ terraform -v
	
4) Install Visual Studio code IDE

5) Download & install AWS cli s/w

6) Create IAM user in AWS & generate access keys

7) Configure IAM user access keys in Environment variables

8) Verify IAM user access keys in cmd

	$ aws configure list



# Step-2 : Launch DevOps Instances

1) Create Key pair in EC2 (name: dpp.pem)

2) Execute terraform script to Create VPC + EC2 instances


# Step-3 : Setup Ansible Server

1) Connect to Ansible VM using pem file 

2) Execute below commands to install ansible 

sudo su -
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible

3) Create hosts file in /opt 
	
	$ cd /opt
	$ vi hosts

4) Add below content in hosts file 

[jenkins-master]
18.209.18.194

[jenkins-master:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/opt/dpp.pem
 
[jenkins-slave]
54.224.107.148

[jenkins-slave:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/opt/dpp.pem
 

5) Upload dpp.pem file to /opt directory and remove write permissions

	$ chmod 400 dpp.pem
 
6) Test the connectivity 

$ ansible -i hosts all -m ping 


# Step-4 : Jenkins Master & Slave Setup 


1) Connect to jenkins master machine and check jenkins status 

	$ sudo su - 
	$ service jenkins status

2) Execute Ansible Playbook to setup Jenkins in Master Node

	- copy jenkins-setup.yaml into /opt/ 
	
	$ ansible-playbook -i hosts jenkins-master-setup.yaml
	
3) Execute Ansible Playbook to setup Maven in Jenkins Slave 

	- copy jenkins-slave-setup.yaml into /opt 

	$ ansible-playbook -i hosts jenkins-slave-setup.yaml
	
	
### Adding Jenkins Slave Node To Master Node ###

4) Add Credentials 

1. Manage Jenkins --> Manage Credentials --> System --> Global credentials --> Add credentials

2. Provide the below info to add credentials   
   kind: `ssh username with private key`  
   Scope: `Global`     
   ID: `maven_slave`    
   Username: `ubuntu`  
   private key: `dpp.pem key content`  

5) Add slave node to master node

   Follow the below setups to add a new slave node to the jenkins 
   
1. Goto Manage Jenkins --> Manage nodes and clouds --> New node --> Permanent Agent    

2. Provide the below info to add the node   
   Number of executors: `3`   
   Remote root directory: `/home/ubuntu/jenkins`  
   Labels: `maven`  
   Usage: `Use this node as much as possible`  
   Launch method: `Launch agents via SSH`  
        Host: `<Private_IP_of_Slave>`  
        Credentials: `<Jenkins_Slave_Credentials>`     
        Host Key Verification Strategy: `Non verifying Verification Strategy`     
   Availability: `Keep this agent online as much as possible`
   

# Step-5 : Create First Jenkins Pipeline Job


1) Create Jenkins Pipeline 

2) Add Build Stage To Pipeline 


# Step-6 : SonarQube Integration


1) Create Sonar cloud account on https://sonarcloud.io

2) Generate an Authentication token on SonarQube 

Account --> my account --> Security --> Generate Tokens

Token : 8122aeaf0aabfa0708813588365299c33181de6a

3) On Jenkins create credentials  

Manage Jenkins --> manage credentials --> system --> Global credentials --> add credentials  - Credentials type: Secret text  - ID: sonarqube-key

4) Install SonarQube plugin    

Manage Jenkins --> Available plugins   -->  Search for sonarqube scanner

5) Configure sonarqube server    

Manage Jenkins --> Configure System --> sonarqube server    Add Sonarqube server    - Name: sonar-server    - Server URL: https://sonarcloud.io/    - Server authentication token: sonarqube-key

6) Configure sonarqube scanner    

Manage Jenkins --> Global Tool configuration --> Sonarqube scanner    Add sonarqube scanner    - Sonarqube scanner: sonar-scanner

7) Write sonar-project.properties file  

sonar.verbose=true
sonar.organization=ashokit
sonar.projectKey=ashokit_instalreels
sonar.projectName=instareels
sonar.language=java
sonar.sourceEncoding=UTF-8
sonar.sources=.
sonar.java.binaries=target/classes
sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml


8) Create Organization and Project in sonarcloud and configure those details in sonar-project.properties file and push sonar-project.properties file to git repo 

9) Add Sonar stage in jenkins pipeline 

https://docs.sonarsource.com/sonarqube/9.8/analyzing-source-code/scanners/jenkins-extension-sonarqube/

		stage('SonarQube analysis') {
            environment{
                scannerHome = tool 'ashokit-sonarqube-scanner'
            }
			
			steps{
				withSonarQubeEnv('ashokit-sonarqube-server') {
					sh "${scannerHome}/bin/sonar-scanner"
				}
			}
        }

11) Create Quality Gate in sonar cloud and make it default


# Step-7 : Jfrog Artifactory Creation


1) Create JFrog Artifactory account (pwd: Jfrogashokit1)

2) Generate an access token with username (username must be your email id)

	Settings --> Platform Configurations --> User Mgmt --> Access Tokens

3) Add username and password under jenkins credentials 

	Manage Jenkins -> Credentials -> Add Credentials -> username with password

4) Install Artifactory plugin

5) Create Docker Artifactory in Jfrog 


# Step-8 : Integrate Docker 


1) Install Docker in jenkins-slave-server using ansible playblook (v2)

2) Install Docker Pipeline Plugin 

3) Configure Docker Build stage in Jenkins Pipeline 

	
		def imageName = 'ashokit.jfrog.io/ashokit-docker-local/insta'
		def version   = '2.1.4'


		stage(" Docker Build ") {
            steps {
                script {
                    echo '<--------------- Docker Build Started --------------->'
                    app = docker.build(imageName+":"+version)
                    echo '<--------------- Docker Build Ends --------------->'
                }   
            }
        }

4) Configure Docker Image Push stage in Jenkins Pipeline 		
        
        stage (" Docker Publish "){
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->'  
                    docker.withRegistry(registry, 'jfrog-cred'){
                        app.push()
                    }    
                    echo '<--------------- Docker Publish Ended --------------->'  
                }
            }
        }

5) Check Jfrog Docker Repository



# Step-9 : Test Docker image by creating container


1) Connect with Jenkins Slave machine 

2) Switch to root user 

	$ sudo su - 
	
3) Execute below commands 

	$ docker images 
	
	$ docker run -d -p 8000:8080 <image-id>
	
	$ docker ps -a 
	
	$ docker logs <container-id>
	
4) Enable 8000 in security group and access application

	URL : http://public-ip:8000/



# Step-10 : Setup Kubernetes Cluster	


1) Setup EKS Cluster using Ansible Playbook (eks, sg_ekgs, vpc)



# Step-11 : Configure EKS in Jenkins Slave


1) Setup EKS Cluster using Ansible Playbook (eks, sg_ekgs, vpc)

Setup kubectl

curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.9/2023-01-11/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin
kubectl version

2) Install AWS CLI 

yum remove awscli 
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --update

3) Configure awscli to connect with aws account

 aws configure
 Provide access_key, secret_key

4) Download Kubernetes credentials and cluster configuration (.kube/config file) from the cluster

 aws eks update-kubeconfig --region us-east-1 --name ashokit-eks-01
 
 

# Step-12 : Execute k8s manifest files 


1) execute ns file 

2) execute deployment file (it will fail)

3) Integrate Jfrog with Kubernetes cluster

Create a dedicated user in jfrog to use for a docker login
user menu --> new user
user name: jfrogcred
email address: ashokit.classes@gmail.com
password: Jfrogashokit1


4) To pull an image from jfrog at the docker level, we should log into jfrog using username and password

 $ docker login https://ashokit.jfrog.io
 
5) genarate encode value for ~/.docker/config.json file

 $ cat ~/.docker/config.json | base64 -w0
 

Note: use above command output in the secret

6) Create secret 




# Step- 13 : Integrate k8s pipeline in jenkins

1) configure aws cli keys for ubunu user 

2) update kube configfile for ubuntu user 

3) Push k8s manifest files into git repo 

4) Add deploy stage in jenkins 

		stage (" Deployment "){
            steps {
                script {
                    echo '<--------------- Deployment Started --------------->'  
                    sh 'sh deploy.sh'
                    echo '<--------------- Deployment Ended --------------->'  
                }
            }
        }


# Step-14 : Helm Setup 


1) install in jenkins-slave 

$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh

2) Validate helm installation 

helm version
helm list

3) Setup helm repo 

helm repo list
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm search repo stable


# Step - 15 : Create Custom HELM Chart


1) Create helm chart 

	$ helm create insta 

2) Replace manifest files in templates directory 

3) Package helm chart 

	$ helm package insta

4) intall helm chart 

	$ helm install insta <pkg name>

5) List helm deployments 

	$ helm list -a 

6) Copy helm chart zip file from slave to our local and push to git hub

7) Add stage in jenkins to install 

	stage(" Deploy ") {
       steps {
         script {
            echo '<--------------- Helm Deploy Started --------------->'
            sh 'helm install insta <chart-file>'
            echo '<--------------- Helm deploy Ends --------------->'
         }
       }
     }



# Step - 16 : Monitoring

### pre-requisites


1. Kubernetes cluster
2. helm

## Setup Prometheus

1. Create a dedicated namespace for prometheus 

   $ kubectl create namespace monitoring
   

2. Add Prometheus helm chart repository

   $ sh helm repo add prometheus-community https://prometheus-community.github.io/helm-charts 
   

3. Update the helm chart repository
   
   helm repo update
   helm repo list
   

4. Install the prometheus

   sh helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring
   

5. Above helm create all services as ClusterIP. To access Prometheus out side of the cluster, we should change the service type load balancer
   
   sh kubectl edit svc prometheus-kube-prometheus-prometheus -n monitoring
   
   
6. Loginto Prometheus dashboard to monitor application
   https://ELB:9090

7. Check for node_load15 executor to check cluster monitoring 

8. We check similar graphs in the Grafana dashboard itself. for that, we should change the service type of Grafana to LoadBalancer

   $ sh kubectl edit svc prometheus-grafana
   

9.  To login to Grafana account, use the below username and password 
    
    username: admin
    password: prom-operator
    
10. Here we should check for "Node Exporter/USE method/Node" and "Node Exporter/USE method/Cluster"
    USE - Utilization, Saturation, Errors
   
11. Even we can check the behavior of each pod, node, and cluster
