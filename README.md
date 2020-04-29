# aws-kops-windows
This tutorial meant to be step by step guide to help people setup their own k8s cluster on AWS ( not EKS )

## Resources 
https://docs.microsoft.com/en-us/windows/wsl/install-win10
https://linuxhint.com/install_aws_cli_ubuntu/
https://kubernetes.io/docs/tasks/tools/install-kubectl/
https://kubernetes.io/docs/setup/production-environment/tools/kops/
https://helm.sh/docs/intro/install/
https://docs.aws.amazon.com/general/latest/gr/rande.html
https://brew.sh/
https://formulae.brew.sh/formula/awscli
https://github.com/jsonmaur/aws-regions
https://github.com/kubernetes/kops/blob/master/docs/cli/kops_create_cluster.md

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
- Add permanent path for Linux:  `echo 'export KOPS_STATE_STORE=s3://clusters.example.com' | sudo tee -a /etc/bash.bashrc`

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

## Creating your first Kubernetes Cluster on AWS
We are going to creating the default cluster ( 1 master and 2 nodes ) \
I compiled a list of KOPs note-worthy options here - https://github.com/howtoclient/aws-kops-instalation/blob/master/kops-options.md

I am going to use the following parameters:
- ``--name cluster.example.com`` - not related to S3 bucket name
- ``--master-size t2.medium`` - I want cluster for testing
- ``--master-zones eu-central-1a`` - single zone for 1 master ( its 1 by default )
- ``--zones  eu-central-1a,eu-central-1b,eu-central-1c `` - I am going with spot instances, multiple regions is recommended

Lets begin:
- Open your Terminal ( linux or otherwise )
- ``kops  create cluster --name=cluster.ratchet.gg --master-size=t2.medium --master-zones=eu-central-1a --zones=eu-central-1a,eu-central-1b,eu-central-1c --yes``
- Cluster deployment will take **10-20 minutes**, check status with ``kops validate cluster``
- You can see the cluster deploying in AWS account in EC2 instances, route 53, Auto Scaling groups, Load balancers etc

NOTES:
- Without `--yes` it will apply the config but wont actually deploy. Its useful for testing new configs but this is a new cluster so i dont need to test.
- In case you want to change settings you can run ``kops delete cluster CLUSTERNAME [--yes]``


