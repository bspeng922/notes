# Openstack subnets

When creating a VM on a network with multiple subnets, you can specify the subnet that the VM attaches to by using the v4-fixed-ip option after the net-id. If these subnets are attached to the same router, VMs on each subnet can communicate with each other, but offer broadcast domain isolation within each subnet. If these subnets are attached to different routers, then the VMs are isolated from each other.

