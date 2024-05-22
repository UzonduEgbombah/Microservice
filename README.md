## 11 MICROSERVICE CICD PIPELINE PROJECT ULTIMATE DEVOPS PIPELINE

#### Microservices architecture has gained popularity in software development for several reasons. Here are a few key advantages:

#### Scalability:

Microservices allow individual components of an application to be scaled independently, ensuring efficient use of resources. This is particularly useful for handling varying loads and optimizing performance.

#### Flexibility and Agility:

Development teams can work on different services simultaneously, allowing for faster iterations and more frequent releases. This also supports diverse technology stacks, as each microservice can be built with the most appropriate technology for its specific function.

#### Fault Isolation:

In a microservices architecture, a failure in one service does not necessarily bring down the entire system. This isolation helps in maintaining overall system stability and resilience.

#### Ease of Deployment:

Microservices can be deployed independently, which simplifies the process of rolling out updates and new features. This reduces the risk of large-scale failures and allows for continuous deployment practices.

#### Improved Maintainability:

With services being smaller and focused on specific tasks, codebases are easier to manage and understand. This modularity simplifies maintenance, debugging, and refactoring processes.

#### Enhanced Testing and Monitoring:

Since each microservice is a discrete unit, it is easier to test and monitor individual services. This leads to more effective detection and resolution of issues.

#### Organizational Alignment:

Microservices align well with modern development teams that follow DevOps and Agile methodologies. They support distributed teams working on different components without stepping on each other's toes.

## Project Implementation

- Setup an EC2 Instance : t2.large (8gb ram), 25gb Storage, and remember to add needed ports to your security group

- Set up your Iam Users and attach policies that would give eks enough priviledge

##### below policies
AmazonEC2FullAccess

AmazonEKS_CNI_Policy

AmazonEKSClusterPolicy	

AmazonEKSWorkerNodePolicy

AWSCloudFormationFullAccess

IAMFullAccess

#### One more policy we need to create with content as below
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "eks:*",
            "Resource": "*"
        }
    ]
}
```
Attach this policy to your user as well

![Policies To Attach](https://github.com/jaiswaladi246/Microservice/blob/Infra-Steps/Policies.png)


## Install The Cli's

# AWSCLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure
```

## KUBECTL

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

## EKSCTL

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/f65d6166-7e54-446d-b54a-9f84d5bb76be)


## Create EKS CLUSTER

```bash
eksctl create cluster --name=EKS-1 \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup

eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster EKS-1 \
    --approve

eksctl create nodegroup --cluster=EKS-1 \
                       --region=us-east-1 \
                       --name=node2 \
                       --node-type=t3.medium \
                       --nodes=3 \
                       --nodes-min=2 \
                       --nodes-max=4 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=devops \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access
```


![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/cba29172-1355-42a2-b105-d606524ce8b5)



![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/90f1760c-b8f0-47ca-aead-489fd19e0c76)



![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/73b2f38d-a6da-4320-8897-7f5e1077a128)




#### Setup Jenkins


```sh
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
```

- Install Java [openjdk -17 headless]
- Install docker and run the command bellow for docker

```sh
sudo chmod 666 /var/run/docker.sock
```

- Paste the jenkins server IP on your browser and login


## Installation Of Plugins For CI/CD

>>>>  /manage jenkins/plugins/available plugins.

##### Install the following:

- Docker

- Docker Pipeline

- Kubernetes / Kubernetes Cli

- Multi branch scan webhook trigger


![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/966e1979-d005-48c5-919a-7c074e40660f)



- / manage jenkins / tools / docker


![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/58553b62-75e4-46e0-a640-2c5c727f97c6)


- / manage jenkins / credentials / global


![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/a6d9e00f-4522-43b2-8eab-68cb51ad496d)


![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/fb31b0f7-63b1-4cf7-913c-12bc39b80c2b)


#### Edit Your Micro-Service JenkinsFile To Match Your Dockerhub Repo

you can find the jenkinsfile in each of the branches in this repository, just edit the dockerhub name into jenkins script
Example:

```sh
pipeline {
    agent any

    stages {
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t uzondu/adservice:latest ."
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push uzondu/adservice:latest "
                    }
                }
            }
        }
    }
}
```

#### Branches

![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/c89c6ffc-5b67-442e-9099-4b3f3bb8291b)


- Create a new item and and name it " MICROSERVICE E-COMMERCE "

- select multi-branch pipeline and create

#### Branch Source

- put in your git url

![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/0007480c-9cbf-4b5b-aa5e-da6c5b9c8ed6)


#### Build Configuration

- Script path "Jenkinsfile"


![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/834bafc8-0393-4659-8f86-79a14c53895e)



#### Scan Multi Branch Pipeline Triggers

- scan webhook

- put in name of choice

- tap on the "?"

- copy url and edit to have the jenkins url and the name chosen earlier


![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/724dc38a-dd71-4aff-aa7e-8dc4a0ee9fc8)


- now go to your github repo settings and select webhook add the edited url gotten from jenkins and add webhook


![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/662db6df-364b-4e28-92ab-4b71df87e9f0)



![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/01280f43-c1f9-4d8a-9b9e-218c7bef1aa1)


#### Pipeline Should Be built Successfully

![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/1a2d2b3a-031e-46ca-a200-25495253ba02)



![](https://github.com/UzonduEgbombah/Microservice/assets/137091610/792514d5-634e-47e7-9528-15edec307402)











































































* Create Servcie account/ROLE/BIND-ROLE/Token
















































