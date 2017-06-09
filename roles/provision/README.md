# Provision

This role create virtual machines on a cloud. If you already have machines to run fabric network,
you can simply skip this playbook.  

## Requirements if use OpenStack cloud

- [Install openstack shade if use OpenStack cloud](http://docs.openstack.org/infra/shade/installation.html)
- [Ubuntu cloud image 16.04 available on your cloud if you use cloud environment] (https://cloud-images.ubuntu.com/releases/16.04/)

If you will be using an Ubuntu system as Ansible controller, ansible openstack cloud
module need to be installed before you can use this role. Use the following scripts
to install needed packages on your Ansible controller. If you have other system as
your Ansible controller, you can do similar steps to setup the environment, the
command may not be exact the same but the steps you need to do should be identical.

    sudo apt-get update

    sudo apt-get install python-dev python-pip libssl-dev libffi-dev -y
    sudo pip install --upgrade pip

    sudo pip install six==1.10.0
    sudo pip install shade==1.16.0

### Prep

#### Deal with ssh keys for access virtual machines

If you do not have a ssh key, then you should create one by using a tool.
An example command to do that is provided below. Once you have a key pair,
ensure your local ssh-agent is running and your ssh key has been added.
This step is required. Not doing this, you will have to manually give
passphrase when script runs, and script can fail. If you really do not want
to deal with passphrase, you can create a key pair without passphrase::

    ssh-keygen -t rsa -f ~/.ssh/interop
    eval $(ssh-agent -s)
    ssh-add ~/.ssh/interop

#### General Openstack Settings

Ansible's OpenStack cloud module is used to provision compute resources
against an OpenStack cloud. Before you run the script, the cloud environment
will have to be specified. Sample files have been provided in vars directory.
If you target ubuntu, you should use vars/ubuntu.yml as the sample, if you
target coreos, you should use vars/coreos.yml file as the sample to create
your own environment file. Here is an example of the file::

    auth: {
      auth_url: "http://x.x.x.x:5000/v3",
      username: "demo",
      password: "{{ password }}",
      domain_name: "default",
      project_name: "demo"
    }

    cluster: {
      target_os: "ubuntu",
      image_name: "Ubuntu 16.04",
      region_name: "",
      availability_zone: "compute_enterprise ",
      validate_certs: True,
      private_net_name: "demonet",
      flavor_name: "m1.medium",
      public_key_file: "/home/ubuntu/.ssh/fd.pub",
      private_key_file: "/home/ubuntu/.ssh/fd",
  
      name_prefix: "fabric",
      # stack_size determines how many virtual or physical machines we will have
      # each machine will be named ${name_prefix}_001 to ${name_prefix}_${stack_size} 
      stack_size: 3,
  
      # If volume want to be used, specify a size in GB, make volume size 0 if wish
      # not to use volume from your cloud
      volume_size: 2,
      # cloud block device name presented on virtual machines.
      block_device_name: "/dev/vdb"
    }

The values of the auth section should be provided by your cloud provider. When
use keystone 2.0 API, you will not need to setup domain name. You can leave
region_name empty if you have just one region. You can also leave
private_net_name empty if your cloud does not support tenant network or you
only have one tenant network. The private_net_name is only needed when you
have multiple tenant networks. validate_certs should be normally set to True
when your cloud uses tls(ssl) and your cloud is not using self signed
certificate. If your cloud is using self signed certificate, then the
certificate can not be easily validated by ansible. You can skip it by setting
the parameter to False. currently the only value available for target_os is
ubuntu and coreos. Supported ubuntu releases are 16.04 and 16.10. Supported
coreos is the stable coreos openstack image.

You should use a network for your OpenStack VMs which will be able to access
internet. For example, in the example above, the parameter private_net_name
was configured as my_tenant_net, this will be a network that all your VMs
will be connected on and the network should have been connected with a router
which routes traffic to external network.

stack_size is set to 3 in the example configuration file, you can change that
to any number you wish, but it must be 2 at minumum. In that case, you will
have one master node and one worker node for fabric cluster. If you set stack_size
to a bigger number, one node will be used as the master, and the rest of the
nodes will be used as worker. Please note that master node will also act as
worker node.

public key and private key files should be created before you run the workload
these keys can be located in any directory that you prefer with read access.

volume_size and block_device_name are the parameter that you can set to allow
the workload script to provision the right size of cinder volume to create
fabric volumes. Each cinder volume will be created, partitioned, formated, and
mounted on each worker and master node. The mount point is /storage. A pod or
service should use hostPath to use the volume.
