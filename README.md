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
