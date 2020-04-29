# Setting up OVPN on KOPs for development
This is not for everyone, generally speaking you can expose any service to localhost but i found it both inconvenient and unstable.
What i would like to do is to have the ability to access internal kubernetes services through a VPN but i dont want all of my traffic going through it.

What i am going to do is setup Open VPN on my cluster in the `default` namespace, and configure the client to only use it for ip's starting with 100.x.x.x with are cluster internal IPs

## First things first Install openvpn client
- Windows - Openvpn https://openvpn.net/
- MacOS - Tunnelblick - https://tunnelblick.net/downloads.html
- Linux - https://openvpn.net/vpn-server-resources/connecting-to-access-server-with-linux/

## Install open vpn on cluster
- ``helm install stable/openvpn --namespace "default"``
  - You can also set it up in its own namespace and use `externalName` to expose services to its namespace
  - You can do that by adding another service that exposes the original service
    - https://github.com/howtoclient/nodejs-hello/blob/master/k8s-template/service-external.yml
- Now we need to configure our partial IP gateway
  - Open Kubernetic and navigate to "Pod Management" -> "config" and open the `openvpn` config
  - Click on "Edit"
  - under `newClientCert.sh` there is a line ``redirect-gateway def1``
  - Remove this line and place ``route 100.0.0.0 255.0.0.0 vpn_gateway`` instead
    - this tells the vpn to only route IPs starting with 100. through the VPN network
    - for more information go to https://github.com/kubernetes/kops/issues/7325
  - scroll all the way down and click "Apply Changes"
  - the full config name you see is your "helm_release" we will need it for the next step.
- To create certificate we need to create a script
  - Open your Terminal ( linux or otherwise )
  - ``nano create-config.sh`` and paste the open vpn script
  - press `ctrl + x` then `y` and then `Enter`
  - ``chmod +x create-config.sh``
  - run ``./create-config.sh my-key default  openvpn-1588201812`` you will probably have to replace `openvpn-1588201812` with your release name from the config
  - run `ls` you will see a file ``my-key.ovpn`` listed
- This is your config file, simply import it into your VPN client
  - In windows the DNS resolver doesnt work as well 
- For Linux in Windows 10 - create new text file called `my-key.ovpn` and save `cat my-key.ovpn` output there.

## Testing vpn connection
- Testing gateway IPs
    - hen going to google.com and searching `my ip` your IP should remain teh same when connected or disconnected from the VPN
- Testing in-cluster connection
    - If you have service running you should be able to connect to it directly, i have `nodejs-hello` 
      - so going to `http://nodejs-hello:3000` on MAC or Linux will return `Hello World!`
      - On windows ( because windows is special ) you can access your internal services with 
        - `http://nodejs-hello.default.svc.cluster.local:3000`

  
## Openvpn script from 
```
#!/bin/bash

if [ $# -ne 3 ]
then
  echo "Usage: $0 <CLIENT_KEY_NAME> <NAMESPACE> <HELM_RELEASE>"
  exit
fi

KEY_NAME=$1
NAMESPACE=$2
HELM_RELEASE=$3
POD_NAME=$(kubectl get pods -n "$NAMESPACE" -l "app=openvpn,release=$HELM_RELEASE" -o jsonpath='{.items[0].metadata.name}')
SERVICE_NAME=$(kubectl get svc -n "$NAMESPACE" -l "app=openvpn,release=$HELM_RELEASE" -o jsonpath='{.items[0].metadata.name}')
SERVICE_IP=$(kubectl get svc -n "$NAMESPACE" "$SERVICE_NAME" -o go-template='{{range $k, $v := (index .status.loadBalancer.ingress 0)}}{{$v}}{{end}}')
kubectl -n "$NAMESPACE" exec -it "$POD_NAME" /etc/openvpn/setup/newClientCert.sh "$KEY_NAME" "$SERVICE_IP"
kubectl -n "$NAMESPACE" exec -it "$POD_NAME" cat "/etc/openvpn/certs/pki/$KEY_NAME.ovpn" > "$KEY_NAME.ovpn"
```