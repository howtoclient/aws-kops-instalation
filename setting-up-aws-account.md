# Setup AWS IAM account and S3 bucket for KOPS
It is necessary for KOPs to control your aws account to create and delete cluster services and security groups

## Setup AWS account and permissions
NOTE: I am going to use Administrator Level access because its easy to setup. This is not recommended for production and you should create user with appropriate permissions later on. \
In general KOPs IAM user requires
- AmazonEC2FullAccess
- AmazonRoute53FullAccess
- AmazonS3FullAccess
- IAMFullAccess
- AmazonVPCFullAccess

Now thats out of the way, lets begin.
Login to your AWS console and navigate to Services -> IAM (https://console.aws.amazon.com/iam/home)
- Click on Users -> Add User
- Name your user `kops` ad tick the "Programmatic access" checkbox.
- Click on `Attach existing policies directly` and search for `AdministratorAccess`. Select it.
- You can skip the tags section
- After you create the user, make sure to save the AWS access ID and secret Key, you wont have another chance to copy them ( or download the csv )
- If you didnt copy the key you will have to generate a new one \

Next navigate to Services S3
- Click on 'Create bucket'
- name the bucket `clusters.example.com` ( it can be any name but it helps if it has meaning )
- Select the region ( NOTE this must be the same region you will be creating your cluster in)
- leave the rest unchanged and click "Create bucket"

Next we need to make sure you have route53 hosted zone, You can either buy a domain or use existing domain through dns
- Navigate to Services -> Route53
- Click on "Hosted Zones"
- Click on Create "Create Hosted Zone"
- Write your existing domain name. Leave the type on "Public Hosted Zone"
- Copy the 4 NS domains ( note that they have extra '.' at the end) and apply them at your domain provider
- I am not going into SSL certificates here \

NOTE: After cluster creating you can switch to Cloudflare or stay with Route53, this is only nessesary for the setup ( Please let me know if im wrong )
