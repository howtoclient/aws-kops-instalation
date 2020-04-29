# Setup tools and environment on Linux ( ubuntu 18 )
I didnt go into depth looking for easier ways, Its the same as windows ubuntu app :)

## resources:
- https://linuxhint.com/install_aws_cli_ubuntu/
- https://kubernetes.io/docs/tasks/tools/install-kubectl/
- https://kubernetes.io/docs/setup/production-environment/tools/kops/
- https://helm.sh/docs/intro/install/
- https://docs.brew.sh/Homebrew-on-Linux

## Step 1: Gather all the tools you need:
- awscli - needed to be able to access our AWS cloude
- kubectl - needed to deploy and update your kubernetes cluster
- kops - used to deploy configure and update your cluster
- helm - used to easily create and deploy services
- restart and verify installationad

## Install aws-cli
Open Linux Terminal and run these code snippets in order
- ``sudo apt-get update``
- ``sudo apt-get install awscli``
- ``aws --version`` - my version for this tutorial is `aws-cli/1.14.44`

## Install kubectl
Open Linux Terminal and run these code snippets in order
- ``cd ~``
- ``curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl``
- ``chmod +x ./kubectl``
- ``sudo mv ./kubectl /usr/local/bin/kubectl``
- ``kubectl version --client`` - my version for this tutorial is `v1.18.2`

## Install kops
Open Linux Terminal and run these code snippets in order
- ``cd ~``
- ``curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64``
- ``chmod +x kops-linux-amd64``
- ``sudo mv kops-linux-amd64 /usr/local/bin/kops``
- ``kops version`` - my version for this tutorial is `1.16.1`

## Install heml
Open Linux Terminal and run these code snippets in order
- ``cd ~``
- ``curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3``
- ``chmod 700 get_helm.sh``
- ``./get_helm.sh``
- ``helm version`` - my version for this tutorial is `v3.2.0`

## Restart and verify
To make sure everything runs properly i highly recommend to restart your PC, enter Linux Terminal and run versions
- ``aws --version`` - my version for this tutorial is `aws-cli/1.14.44`
- ``kubectl version --client`` - my version for this tutorial is `v1.18.2`
- ``kops version`` - my version for this tutorial is `1.16.1`
- ``helm version`` - my version for this tutorial is `v3.2.0`


### Alternatively you can install Homebrew 
( dont forget to configure paths ) and use that to install https://docs.brew.sh/Homebrew-on-Linux
- ``brew install awscli``
- ``brew install kubectl``
- ``brew install kops``
- ``brew install heml``

Congrats! Now you have linux running on your Windows machine and you can move to the next step!