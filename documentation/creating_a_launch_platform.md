# Setting Up a Launch Development Platform

## Introduction

A launch platform is required to perform the basic tasks of creating and configuring the cluster. I want to do everything in the cloud so I'm starting by writing my documentation directly into GitHub.

## Objectives

Here I will document the steps to create a development environment in the cloud (I want to if possible design everything to be cloud-based and not rely on configuring or using local personal resouces - that way my entire approach to learning is standardized and not dependent on any platform).

## Launch Platform

You have to start somewhere - I want to start with a VM that can be configured (and automated) to be ready to provision the cloud native infrastructure (Google, AWS, OCI, Azure, etc). I want to eventually automate this - but I will start with documentation of the steps to provision this platform.

## Requirements

## Setup Steps

### 1. Instance Creation
1. Create a VM that will serve as the launch/development platform - this is where development and automation happens based on this repo
2. I'm creating an Ubuntu 20.04 instance on Digital Ocean (starting with the lowest cost plan)
3. Select the closest datacenter region
4. Don't bother with ssh keys - you will be working entirely in the cloud so that your dev platform can be utilized through the cloud shell on the cloud provider portal
5. You can run init scripts on the VM when it is created - this is how I plan to work with this instance in the future, but for now I will capture the manual steps and test as a shell script until it is completed (Rule #1: Do Once, Test your steps, then automate)
6. Hostname and tags are unimportant
7. For now I will start/stop the instance - but in the future I plan to be able to automatically recreate it when needed (IaC)

### 2. Instance Configuration
Sources: https://linuxconfig.org/how-to-update-ubuntu-packages-on-ubuntu-20-04-focal-fossa-linux

1. If you haven't already, create or start the instance
2. Sign on as root from the cloud provider console
3. Update the packages index list: apt update
4. Look at packages due for update: apt list --upgradable
5. Upgrade packages: apt upgrade -y
6. Update kept back packages: apt dist-upgrade -y
7. Remove unneeded packages: apt autoremove -y
8. Reboot the instance: poweroff then restart from the console

### 3. Installing Git
Sources: https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-20-04
1. Check if already installed: git --version
2. I have 2.25.1 - no need to make changes

### 4. Installing Terraform
Reference: https://cloudlinuxtech.com/install-terraform-on-ubuntu-uninstall-terraform/
apt upgrade
apt install terraform
terraform -v # currently 1.1.7
```
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
apt-add-repository "deb [arch=$(dpkg --print-architecture)] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
apt update
```

### 5. Installing Amazon Cloud CLI
References: 
- https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
- https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html
```
apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
./aws/install
aws --version # currently 2.5.2
```

## 6. Installing kubectl
```
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubectl
kubectl version
```

### 7. Generating a new SSH Key
References: 
- https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
- https://superuser.com/questions/478798/running-ssh-keygen-without-human-interaction
```
ssh-keygen -t rsa -b 4096 -q -f "$HOME/.ssh/id_rsa" -N ""
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```
- Add the public ssh key to your github account


### 8. Clone the Terraform Repository
```
git clone git@github.com:NickBaynham/sre-terraform.git
```

## 9. Git Configuration to push changes
```
 export git_user="Your Name"
 export git_email="you@example.com" # paraterized setup scripts
 git config --global user.email "$git_email"
 git config --global user.name "$git_user"
 git push
```

## 10. Provision Terraform State for each Cluster
- Execute the steps in: https://github.com/NickBaynham/sre-terraform/blob/main/shared_state/README.md

## 11. Provision the Cluster Network
- Execute the steps in: https://github.com/NickBaynham/sre-terraform/tree/main/clusters-vpc

## 12. Provision the Cluster Infrastructure
- Execute the steps in: https://github.com/NickBaynham/sre-terraform/blob/main/clusters/README.md

## 13. Install and Configure AWS CLI
```
apt install awscli -y
aws --version
aws configure
```
## 14. Retrieve the cluster kubeconfig file
```
aws eks --region $(terraform output aws_region) update-kubeconfig --name $(terraform output cluster_full_name)
terraform output authconfig | kubectl -n kube-system create -f -
kubectl get ns
```
## 15. Cleaning Up and destroying infrastructure resources
```
cd ~/sre-terraform/clusters
terraform init
terraform destroy

cd ~/sre-terraform/clusters-vpc
terraform init
terraform destroy

# if you need to destory shared state:
aws s3 rm s3://nbaynham-terraform_state --recursive
aws s3 rm s3://nbaynham-vpc-terraform-state --recursive

aws s3api delete-object --bucket nbaynham-terraform-state --key nbaynham-terraform-state
aws s3api delete-object --bucket nbaynham-vpc-terraform-state --key nbaynham-vpc-terraform-state

aws s3 rb s3://nbaynham-terraform-state --force
aws s3 rb s3://nbaynham-vpc-terraform-state --force

cd ~/sre-terraform/shared_state
terraform init
terraform destroy

```
Note: You should verify on the AWS console that destroy has taken effect correctly

## 16. Installing Required Tools for Ansible Configuration Management
```
apt-get update
apt-get install python3 -y
apt-get install python3-pip -y
pip3 install virtualenv
```
## Installing Packer
```
apt-get update && apt-get install packer -y
```
