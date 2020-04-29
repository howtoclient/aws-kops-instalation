# Setting up kubernetic on Ubuntu windows app
Since we are using a subsystem, we need to direct our kubernetic config to take from the Ubuntu app
There are 2 main ways to do so:

## Copy kubenretic config to a file
The easy way
- Run ``cat ~/.kube/config`` copy the contents into text file
- Go to Kubernetic -> preferences and place the path to your config file

The weird way:
- On windows, open file explorer
- Navigate to %userprofile%\AppData\Local\Packages
- Place ubuntu18 in search bar, you should see a folder
- Navigate to LocalState -> rootfs -> home -> your user name -> .kube  you should see a config file
- Go to Kubernetic -> preferences and place the path to this config file ( make sure you dont route to a folder )

NOTE: you will most likely need to close and open the Kubernetic app, You know that it works when you see Nodes > 0