# aws-kops-windows
This tutorial meant to be step by step guide to help people setup their own k8s cluster on AWS ( not EKS )

## First things first, setup your environment
We will need:
- aws cli
- kubectl
- kops
- helm

You can follow one of those instructions for your platform, or install yourself.
- Windows - https://github.com/howtoclient/aws-kops-instalation/blob/master/setting-up-windows.md
- Linux - https://github.com/howtoclient/aws-kops-instalation/blob/master/setting-up-linux.md
- MacOS - https://github.com/howtoclient/aws-kops-instalation/blob/master/setup-mac-os.md

## Setup AWS account and permissions
NOTE: I am going to use Administrator Level access because its easy to setup. This is not recommended for production and you should create user with appropriate permissions later on. \
Create IAM programmatic account with the following permissions:
- AmazonEC2FullAccess
- AmazonRoute53FullAccess
- AmazonS3FullAccess
- IAMFullAccess
- AmazonVPCFullAccess

Create S3 bucket that KOPs will use for cluster state (`clusters.example.com`)

If you dont know how please go to
- https://github.com/howtoclient/aws-kops-instalation/blob/master/setting-up-aws-account.md

## Configuring AWS and KOPs on your machine
Now lets make sure we have the right access and state store for our deployment. \
Configure KOPs Store
- Open your Terminal ( linux or otherwise )
- run `export KOPS_STATE_STORE=s3://clusters.example.com`
- Add permanent path for Linux:  `echo 'export KOPS_STATE_STORE=s3://clusters.example.com' | sudo tee -a /etc/bash.bashrc` \

Configure aws cli
- Open your Terminal ( linux or otherwise )
- `aws configure` \
   `Access key ID = KOPs IAM user access key ID` \
   `Access key secret = KOPs IAM user access key secret` \
   `Default region = Regiou of your S3 bucket, this is where we will create our k8s Cluster` \
   `Default output format = json`  
- run `aws iam list-users` to see that everything is working, you should see at least one user.

Create SSH key ( needed for KOPs connection to cluster )
- Open your Terminal ( linux or otherwise )
- `ssh-keygen -t rsa -b 4096 -C "your@email.com"` \
Replace the email with your email, click enter until its generated \
NOTE: If you changed default public key url you will ahve to use `--ssh-public-key ~/path` when creating the k8s Cluster

