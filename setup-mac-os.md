# Setup tools and environment on Mac OS
Easiest thing i ever done. No seriously...

## resources:
- https://brew.sh/
- https://formulae.brew.sh/formula/awscli
- https://kubernetes.io/docs/tasks/tools/install-kubectl/
- https://kubernetes.io/docs/setup/production-environment/tools/kops/
- https://helm.sh/docs/intro/install/

## Step 1: Gather all the tools you need:
- brew - used to install all the other apps
- awscli - needed to be able to access our AWS cloude
- kubectl - needed to deploy and update your kubernetes cluster
- kops - used to deploy configure and update your cluster
- helm - used to easily create and deploy services

## Install homebrew
- ``/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"``
- ``brew -v`` - my version for this tutorial is `v2.2.13`\
 

## Install aws-cli
- ``brew install awscli``
- ``aws --version`` - my version for this tutorial is `aws-cli/2/0.10`

## Install kubectl
- ``brew install kubectl``
- ``kubectl version`` - my version for this tutorial is `v1.18.2`

## Install kops
- ``brew install kops``
- ``kops version`` - my version for this tutorial is `v1.16.1`

## Install helm
- ``brew install heml``
- ``helm version`` - my version for this totorial is `v3.1.2`

That's it, you are done!