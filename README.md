# Jenkins Master–Slave (Agent) Setup on AWS

WHY Jenkins?

Jenkins is an open-source CI/CD tool.
It automates build, test, and deployment.
It reduces manual errors and speeds up delivery.
On code push to GitHub, Jenkins pulls, builds, tests, and deploys automatically.

WHY Jenkins Master–Slave Architecture?

 In real projects, many builds run at the same time.
 A single Jenkins server becomes slow and unstable.
 The Master manages jobs, UI, and scheduling.
 Slaves (Agents) perform heavy build and test tasks.
 Builds run in parallel on multiple agents.
 Result: faster builds, better performance, and high availability.
 

<img width="553" height="104" alt="image" src="https://github.com/user-attachments/assets/a5854a78-f105-4f3d-85a7-6ac0bd918b64" />

2. AWS Architecture Overview
2.1 Components Used

AWS EC2 – For Jenkins Master and Slave

Amazon VPC – Networking

Security Groups – Firewall rules

Key Pair – SSH access

Elastic IP (optional) – Stable access to Jenkins UI

📸 Screenshot Hint:
Add AWS architecture diagram showing Master and Slave EC2 instances.

3. Prerequisites

AWS Account

IAM User with EC2 access

Key Pair (.pem file)

Basic Linux & Jenkins knowledge

📸 Screenshot Hint:
Add AWS Console → EC2 Dashboard screenshot.

4. Step 1: Create EC2 Instance for Jenkins Master
4.1 Launch EC2 Instance

AMI: Amazon Linux 2 or  ubuntu

Instance Type: t2.micro (Free Tier)

Storage: 8–10 GB

Key Pair: Create or select existing

Security Group:

Port 22 – SSH

Port 8080 – Jenkins UI

<img width="553" height="166" alt="image" src="https://github.com/user-attachments/assets/5a81d7ce-259b-42a0-afa0-c14245235a29" />


4.2 Connect to Master EC2
ssh -i key.pem ec2-user@<master-public-ip>

or you can direclty onnect ec2 with "ec2 instance connect from aws console"
<img width="492" height="173" alt="image" src="https://github.com/user-attachments/assets/23784abc-14b6-42fb-9321-804b64fa1e25" />





4.3 Install Java & Jenkins 

 from jenkins offical site -- https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

 or 

sudo yum update -y
sudo amazon-linux-extras install java-openjdk11 -y
sudo yum install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins

check jenkins is running :
sudo systemctl status jenkins

<img width="700" height="163" alt="image" src="https://github.com/user-attachments/assets/5f8e282b-65ff-46ac-9082-aacfe79c7389" />


5. Step 2: Access Jenkins Master

Open browser:

http://<master-public-ip>:8080


Get initial admin password:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword


<img width="496" height="198" alt="image" src="https://github.com/user-attachments/assets/d210d905-bb77-420a-8e56-31cdd9dc7ecb" />


6. Step 3: Create EC2 Instance for Jenkins Slave
6.1 Launch Slave EC2

AMI: Amazon Linux 2 or Ubuntu
Instance Type: t2.micro
Same VPC as Master
Security Group:
Allow SSH from Master Security Group or add same security group of master 



6.2 Prepare Slave Machine

install java version for jenkin portal : https://www.jenkins.io/doc/book/installing/linux/#debianubuntu


sudo yum update -y
sudo amazon-linux-extras install java-openjdk11 -y
useradd jenkins
passwd jenkins


check java version : $ java -version

7. Step 4: Configure Jenkins Slave from Master
7.1 Add New Node

<img width="553" height="163" alt="image" src="https://github.com/user-attachments/assets/84d34dd1-0b28-45f6-88b4-e8d000e86bac" />


Manage Jenkins → Manage Nodes and Clouds

Click New Node

<img width="554" height="103" alt="image" src="https://github.com/user-attachments/assets/3c2d172c-686a-418a-a02f-a8c23465cca9" />


7.2 Node Configuration
Field	Value
Node Name	: slave (any name)
Type	:Permanent Agent
Remote : Root Directory	/home/jenkins
Labels:	dev 
Usage	Use this node as much as possible
Launch Method	Launch agent via SSH

<img width="554" height="290" alt="image" src="https://github.com/user-attachments/assets/b4d05d99-886a-4303-a4d7-8d14e5a8aa95" />



7.3 Add SSH Credentials

Host: Slave-server-Private-IP

Now click on Add credentials

Kind: SSH Username with Private Key

Username: ec2-user(for aws linux machine) or ubuntu (for ubuntu machine)

Now add the Private Key (PEM file)

<img width="554" height="313" alt="image" src="https://github.com/user-attachments/assets/1c0b00ca-b43a-402c-9c01-ad4531e79e4f" />

<img width="554" height="329" alt="image" src="https://github.com/user-attachments/assets/a0a06a32-f952-422f-973b-b301ae0160c8" />

<img width="553" height="287" alt="image" src="https://github.com/user-attachments/assets/d81f7326-138d-4ac6-b08b-868f73b52254" />



7.4 Verify Connection

Click Save

Node status should be Connected 
(for reference)

<img width="563" height="257" alt="image" src="https://github.com/user-attachments/assets/f478f66a-2f92-4c7d-82cd-b358c9fa47fa" />


8. Step 5: Run Jenkins Job on AWS Slave
8.1 Create Job

New Item → Freestyle Project

Restrict where this project can be run

Label: dev

<img width="1960" height="1318" alt="image" src="https://github.com/user-attachments/assets/196cac09-5089-4262-936b-a31cb20f752a" />


<img width="2880" height="1434" alt="image" src="https://github.com/user-attachments/assets/48b4dcd4-da59-4716-9b56-c68d63d9067f" />
<img width="1350" height="1110" alt="image" src="https://github.com/user-attachments/assets/0ee49113-29b4-426e-8074-9450e74387d9" />

8.2 Build Execution

Trigger build

Verify execution on slave, before that make sure your master build  executor count should be zero . and use labeling to on slave node.

<img width="554" height="240" alt="image" src="https://github.com/user-attachments/assets/97576a7b-b0f4-4b2d-b09d-f37bb716f7dc" />


9. Communication Flow (AWS)

Jenkins Master schedules job

Master connects to Slave via SSH

Slave executes job

Logs and results sent back to Master

ii) with docker installation in slave node to run docker containers :

cmmds to run on slave :
https://docs.docker.com/engine/install/ubuntu/

in slave instacne after installation :

 docker -v
   24  docker ps
   25  systemctl status docker
   26  docker ps
   27  usermod -aG docker ubuntu
   28  sudo usermod -aG docker ubuntu
   29  docker ps
   30  exit
     docker ps
   32  cat /etc/passwd
   33  groups ubuntu
   34  sudo usermod -aG docker jenkins
   35  ps -ef | grep jenkins
   36  sudo -u ubuntu docker ps
   37  ls -l /var/run/docker.sock
   38  sudo usermod -aG docker ubuntu
   39  sudo reboot

<img width="553" height="279" alt="image" src="https://github.com/user-attachments/assets/3be6fec7-0ef6-4840-9379-168e20b63277" />


now create a pipeline  :

<img width="553" height="253" alt="image" src="https://github.com/user-attachments/assets/6bc82ead-42b0-45cc-8556-d6d3522d82ed" />

add script:

scripts:

pipeline {
    agent { label 'dev' }
    stages {
        stage('Docker') {
            steps {
                sh 'whoami'
                sh 'docker run  hello-world'
            }
        }
    }
}

<img width="553" height="279" alt="image" src="https://github.com/user-attachments/assets/cb08b6bd-334a-42d4-bc68-908a52bd70e3" />

run the pipeline::

<img width="554" height="299" alt="image" src="https://github.com/user-attachments/assets/f723c182-2065-4132-9006-3e0b29333431" />

to check the output console:

<img width="497" height="509" alt="image" src="https://github.com/user-attachments/assets/99f487bc-9abf-4b9a-ba56-f0ee7853c494" />
<img width="554" height="279" alt="image" src="https://github.com/user-attachments/assets/146897f5-3547-4b5e-bb86-df4506006755" />

Issues:
1. if you run on master or missed labling part (to run on slave) , it will be in pending state:

<img width="554" height="264" alt="image" src="https://github.com/user-attachments/assets/71bcd3fc-93e8-4fa2-b898-6a15e2ac3e80" />



after completion of all activites :

terminate the aws instances :

<img width="531" height="306" alt="image" src="https://github.com/user-attachments/assets/5e3ee898-a8b6-47e0-8b1c-fd756e862832" />



10. Advantages of Jenkins on AWS

Auto-scalable infrastructure

High availability

Cost-efficient (pay-as-you-go)

Easy integration with AWS services

Supports distributed builds

11. Common Issues & Troubleshooting
Issue	Cause	Fix
Agent offline	Security group	Allow SSH
Permission denied	Wrong key	Verify credentials
Job runs on master	Missing label	Set node restriction


12. Conclusion

Deploying Jenkins Master–Slave architecture on AWS provides a scalable and secure CI/CD environment. Using EC2 instances for controllers and agents ensures better performance, flexibility, and resource management.
