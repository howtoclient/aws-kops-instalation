# Setting up spot instances on KOPs and AWS
You dont have to use spot instances, and you dont need to have all of your Cluster nodes as spot instnaces but they are 1/3 the price and are very useful because of that. Lets set them up!

NOTE: I am assuming you just created your cluster and dont manage multple clusters so i dont include `--name` in the code examples. If you do have multiple clusters make sure to add it to target a specific cluster


## Creating spot instance group
We need to create a new instance group that KOPs will recognize so we will use `kops` for that
- Open your Terminal ( linux or otherwise )
- Get current instance groups list with ``kops get ig nodes`` You should see your standard instance group for `nodes`
- Before we continue, creating a new instance group will land you into VIM editor. 
  - When editing pay close attention to spacing!
- run ``kops create ig spot-ig``
  - Now we need to edit our configurations ( press `insert` to start editing )
  - under `machineType:` add `maxPrice: "0.0170"`  ( this is for `t2.medium`)
    - https://aws.amazon.com/ec2/spot/pricing/ for price reference
    - NOTE: you dont want to put exact number here, little bit higher is ok
  - set `maxSize` to 4 ( or higher if you want )
  - set `minSize` to 1 ( autoscaler will put more when necessary)
  - under `nodeLabels:` add `on-demand: "false"`
  - press `ecq` write `:wq`
- check that the new instance group is there by running ``kops get ig``
  - if you need to edit use ``kops edit ig spot-ig``
  - you can reduce `on-demand` instances to 1
- Apply changes to your cluster ``kops update cluster --yes``
- verify by going to AWS, navigate to "Services" -> "EC2" -> "Spot Requests"

## Setting up termination handler
When spot instances are stopping they give out a 2 minute notice and we need to make sure that we are able to replace the pods that are running on that node quickly
- ``helm install stable/k8s-spot-termination-handler --namespace kube-system -g``
- check with ``kubectl --namespace=kube-system get pods -l "app.kubernetes.io/name=k8s-spot-termination-handler"``

NOTE: If you dont see the need for auto-scaler you can setup your wanted nodes number and stop here.

## Optional - 100% spot instance nodes  
Kops master must be on-demand but nodes dont have to be as long as you are not running applications that need it and though its not recommended to use only spot instances for my development cycle i can allow my self
- get kops instance groups ``kops get ig``
- delete `on-demand` instance group ``kops delete ig nodes``
- check that they are deleted ``kops get ig``
- Apply changes to your cluster ``kops update cluster --yes``

## Creating autoscaler for our spot instance needs
Well we have spot instances with between 1 and 4 but how will kops know when to add more? Autoscaler.
- first we need to create a policy for the autoscaler to allow it to scale up and down
  - Login to your AWS account and navigate to "Services" -> IAM -> Policies
  - Click on "Create Policy" and select JSON then paste this code 
    - https://github.com/howtoclient/aws-kops-instalation/blob/master/auto-scaler-policy.json
  - then give the policy name `autoscaler.cluster.example.com` and click create
- Navigate to Roles and find `nodes.cluster.example.com` role and open it
- Now click on "Attach policies"
- find our `autoscaler.cluster.example.com` policy, select and click "Attach policy"
- Now lets install the basic autoscaler
  - We need to setup the node name in this yaml first and setup our node group name
    - Login to your aws console and navigate to "Services" -> EC2 -> "Auto Scaling Groups"
    - Copy the name of our `spot-ig` group
  - Open terminal
    - ``wget https://raw.githubusercontent.com/howtoclient/aws-kops-instalation/master/kops-auto-scaler.yml``
    - ``nano kops-auto-scaler.yml`` and at the bottom change `spot-ig.cluster.example.com` to your auto scale group name
    - do `ctrl + x` then `y` and hit `Enter`
  - now run ``kubectl apply -f kops-auto-scaler.yml``
  
## Testing auto scaler
**NOTE: Sometimes your aws account might get a limit of 1 spot instance ( this was my case ) and you will have to contact the aws support team.
You can see if this is the case in Services -> EC2 -> Auto Scaling Groups -> spot-ig under Activity History**

Now lets bring in the heat! Scale our service to 60 \
NOTE:  scaling with spot instances might take a few minutes
- ``kubectl scale deployment nodejs-hello --replicas=60``
  - to restore run - ``kubectl scale deployment nodejs-hello --replicas=2``
- While this is running you can see in aws console that the instance group Desired instances count changed to  more than 1
- Wait for it to stabilize and then set replicas to 2 to check scaling down

## Spot instance yml config example
```yaml
apiVersion: kops.k8s.io/v1alpha2
kind: InstanceGroup
metadata:
  creationTimestamp: null
  name: spot-ig
spec:
  image: kope.io/k8s-1.16-debian-stretch-amd64-hvm-ebs-2020-01-17
  machineType: t2.medium
  maxPrice: "0.0170"                                                                                                                                             maxSize: 4                                                                                                                                                     minSize: 1                                                                                                                                                     nodeLabels:
    kops.k8s.io/instancegroup: spot-ig
    on-demand: "false"                                                                                                                                           role: Node
  subnets:
  - eu-central-1a
  - eu-central-1b
  - eu-central-1c   
```