# KOPs auto deployment with BitBucket
This is my take on a CI-CD deployment that is accessible for developers but is still automatic and based on branch witch allows to have multiple environments.

NOTE: I am assuming that you are familiar with GIT and how to work with it. In this tutorial we weill be using nodejs since its very simple to setup.
- https://github.com/howtoclient/nodejs-hello

## Resources 
The list is very long, please see
- https://github.com/howtoclient/aws-kops-instalation/blob/master/resources.md

## Install nodejs
Install nodejs ( i use 12.x.x LTS version ) https://nodejs.org/en/

## Create Bit Bucket Account and nodejs-hello Repository 
Well first you need to create a bitbucket account, simply follow the steps at https://bitbucket.org/
Once you are inside navigate to "Repositories" ( skip if you already have a repository but for learning purposes i recommend to create a new one )
- Click on "Create repository"
- Name the repository `nodejs-hello` then click "Create"
- Click on "Create .gitignore" and add `node_moduels/` inside and click commit
  - Bitbucket is weird and creates repositories without master branch, this makes master for us

## Clone and configure the nodejs-hello project
If you are familiar with nodejs you can use any existing project, just pay attention to the project and repository name
- Clone the project from Bitbucket repository
- Open the project folder with terminal ( or use built in terminal in your code editor - i am using webstorm )
- init the repository and install express
- ``npm init`` press enters until its done, no need to change anything
- ``npm i express --save`` - installs express-js library and saves to `package.json`
- Now create file called `index.js`
- Then copy this code sample https://expressjs.com/en/starter/hello-world.html
- Now go to `package.json` find “scripts” and add this line under “test: ” add
  - ``"start": "node index.js"``
- now run `npm start`, you should be able to see result on `locahost:3000` 
- Commit and push your changes to `master` branch

## Create Elastic Container Repository ( ECR )
I wanted to use AWS repository because its internal and because it scans the created images. The downside is that we cant auto-create a repository unless we install aws-cli during the pipeline run, you can do that if you like \
NOTE: AWS recommends using repository per image, this is what we are going to do here.
- Login to your aws console
- Navigate to Services -> Elastic Container Registry
- Click on "Create a repository"
- It is important to call the repository with our image name, in our case its ``nodejs-hello``
  - You can enable the scan option while you are here
- save the repository URL somewhere
  
## Configure BitBucket repository
Now lets configure our BitBucket repository to support our deployment
- Login to https://bitbucket.org/
- Navigate to `nodejs-hello` repository
- Navigate to "Repository settings" -> "Deployments"
- Open the "Production" deployment
- **When configuring the variables Tick the `Secure` checkbox so no one can see them**
- Add aws keys you saved from creating the IAM user here
  - `AWS_ACCESS_KEY_ID` = your key id
  - `AWS_SECRET_ACCESS_KEY` = your key secret
  - `AWS_DEFAULT_REGION` = your kubernetes region
- Add `KUBE_CONFIG` variable
  - we need our cube configuration here
  - Open terminal and run ``cat ~/.kube/config | base64``
  - Copy and paste this value in the `KUBE_CONFIG` value  
- Navigate to "Repository settings" -> "PIPELINES" -> settings and turn on "Enable Pipelines"
  
## Configure auto deploy pipeline in BitBucket
We did all the configurations we can setup our pipeline and deployment
I added some explanations on the yamls here https://github.com/howtoclient/nodejs-hello
We will create the following files:
- `Dockerfile` - used to run the code inside the image
- `bitbucket-pipelines.yml` - used to tell bitbucket what to do
- `k8s/deployment.yml` - tells kubectl about the deployment of the image
- `k8s/service.yml` - tells kubectl about the service specifications of the app
- `k8s/ingress.yml` - tells ingress-nginx about the service routing ( dont use if service is not exposed )

#### Lets begin
- Create file `Dockerfile` put this snippet and save
  - https://github.com/howtoclient/nodejs-hello/blob/master/Dockerfile
- Create `bitbucket-pipelines.yml` put this snippet and 
  - https://github.com/howtoclient/nodejs-hello/blob/master/bitbucket-pipelines.yml
  - **replace `$AWS_ECR_HOST/nodejs-hello` with your ECR repository URL**
- Create `k8s` folder ( its going to be used for kubectl deployment )
  - Create `k8s/deployment.yml` put this snippet and  
    - https://github.com/howtoclient/nodejs-hello/blob/master/k8s/deployment.yml
    - **replace `[Your ECR host name/nodejs-hello]` with your ECR repository URL**
    - We need to create the secret while we are here
      - Open Kubernetic app
      - Nanigate to "Pod Management" -> "Secrets"
      - Click "+ Create" and name your secret the same you did in the `deployment.yml`
      - Add environment variable ``NODE_ENV=dev``
      - Click `Create`. This is where you are going to place environtment variables for your app
      - NOTE: kubectl requires the values to be `base64` encoded, if you see gibberish this is why.
  - Create `k8s/ingress.yml` and put this code snippet and 
      - https://github.com/howtoclient/nodejs-hello/blob/master/k8s/ingress.yaml
      - **replace `example.com` with your domain**    
  - Create `k8s/service.yml` and put this code snippet
      - https://github.com/howtoclient/nodejs-hello/blob/master/k8s/service.yml
- Commit and push
- Open Kubernetic and navigate to PODs
- Open Bitbucket `nodejs-hello` repository and navigate to `pipelines`
- Watch it run!

## See the restults
Simply navigate to `nodejs-hello.example.com` ( replace example.com with your domain )

If you see hellow-world! you are DONE!

## Verifiying auto-deploy
To make sure that everything is running, change `Hello World!` in `index.js` to something else then commit and push
If after 2-5 mins you see the new workds in  `nodejs-hello.example.com` ( replace example.com with your domain ) your auto deployment is working.



