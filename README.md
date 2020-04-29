# aws-kops-windows
This tutorial meant to be step by step guide to help people setup their own k8s cluster on AWS ( not EKS )

## Resources 
The list is very long, please see \
- https://github.com/howtoclient/aws-kops-instalation/blob/master/resources.md

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
I compiled a list of KOPs note-worthy options here:
- https://github.com/howtoclient/aws-kops-instalation/blob/master/kops-options.md

I am going to use the following parameters:
- ``--name cluster.example.com`` - not related to S3 bucket name
- ``--master-size t2.medium`` - I want cluster for testing
- ``--master-zones eu-central-1a`` - single zone for 1 master ( its 1 by default )
- ``--zones  eu-central-1a,eu-central-1b,eu-central-1c `` - I am going with spot instances, multiple regions is recommended

Lets begin:
- Open your Terminal ( linux or otherwise )
- ```kops  create cluster --name=cluster.ratchet.gg --master-size=t2.medium --master-zones=eu-central-1a --zones=eu-central-1a,eu-central-1b,eu-central-1c --yes```
- Cluster deployment will take **10-20 minutes**, check status with ``kops validate cluster``. Its ready only when all the errors are gone.
- You can see the cluster deploying in AWS account in EC2 instances, route 53, Auto Scaling groups, Load balancers etc

NOTES:
- Without `--yes` it will apply the config but wont actually deploy. Its useful for testing new configs but this is a new cluster so i dont need to test.
- In case you want to change settings you can run ``kops delete cluster CLUSTERNAME [--yes]``

## Setting up Kubernetic
A very useful GUI tool for kubernetes that we will be using to setup our auto-deploy example
Download the tool from https://kubernetic.com/, it should start right up.
- If you are using Unbuntu app on windows please see:
  - https://github.com/howtoclient/aws-kops-instalation/blob/master/kubernetic-linux-windows.md

## Setting up Ingress-nginx on your Cluster
To easily expose services and connect sub-domains i am going to use nginx-ingress

- Open your Terminal ( linux or otherwise )
- ``kubectl apply -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/ingress-nginx/v1.6.0.yaml``
- Go to aws console. Navigate to Services -> EC2 -> Load Balancers, You should see a load balancer there
- Copy the "DNS name"
- Navigate to Services -> Route53 -> Hosted Zones. Click on your hosted zone
- Add two A records for `example.com` and `*.example.com`
  - click on "Create Record Set"
  - for main domain leave Name empty ( for `example.com` )
  - Select Alias - yes
  - Paste the Load balancer URL you copied
  - Click Create
  - Repeat for `*.example.com` ( set Name = `*` )
- Alternatively you can point your Domain to Cloud Flare and add the `A records` for `example.com` and `*.example.com` there.
  
## Conclusion and next steps
This is the easiest way i found to setup KOPs based k8s cluster and configure it to work withing a short amount of time and with a minimal expenses.

So whats next?
There are 3 more steps for getting this cluster not just running but also useful for development and cheap to run, since the cluster is setup order of execution doesnt really matter and you can also skip steps you dont think are necessary
- #### Auto CI-CD deployment
  - https://github.com/howtoclient/aws-kops-instalation/blob/master/kops-auto-deployment-with-bitbucket.md
- #### Spot instances
   - https://github.com/howtoclient/aws-kops-instalation/blob/master/setting-up-spot-instances.md
- #### Access internal services through VPN 
   - https://github.com/howtoclient/aws-kops-instalation/blob/master/setting-up-ovpn-on-kops.md

I hope it helps to get started with K8s at its full potential. I will update this article with my monthly bill for this type of Cluster.