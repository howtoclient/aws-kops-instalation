#KOPS options worth mentioning
He is a list of KOPs options i found note-worthy, See the full list of options in the Resources link

## Resources
- https://github.com/kubernetes/kops/blob/master/docs/cli/kops_create_cluster.md
- https://github.com/jsonmaur/aws-regions

## Kops options you might want to use for this tutorial
- ``--name my-cluster.dev.example.com`` - If you are not using private cluster this name MUST contain the domain configured in Route53
- ``--master-size t2.medium`` - I am using t2.medium for this tutorial. Recommended size is 4 cores 8GB or higher
- ``--master-count 1`` - I am using one but for production you want to use a minimum of 2 ( 3 is recommended )
- ``--master-zones eu-west-1a`` - Notice the `a` at the end of the zone? Kops uses sub-zones.
- ``--node-size t2.medium`` - Nodes that the Pods will use for deployment ( t2.medium is the default )
- ``--node-count 2`` - Number of nodes created by default, ( Doesnt really matter if you plan to switch to spots with auto scale later)
- ``--zones eu-west-1a,eu-west-1b,eu-west-1c`` -  Notice the `a,b,c` at the end of the zone? Kops uses sub-zones. I am planning to move to spot instances and more zones = more spots to choose from.
- ``--api-loadbalancer-type public`` - Use this if you are setting up multiple Master Nodes, i am using one for this tutorial.
- Sub-Zone list - https://github.com/jsonmaur/aws-regions
- Full options list - https://github.com/kubernetes/kops/blob/master/docs/cli/kops_create_cluster.md