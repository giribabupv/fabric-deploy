# Hyperledger Fabric Deployment

This ansible playbook project will accomplish the following tasks

 - Provision virtual machine nodes to participate in fabric network
 - Build hyperledger fabric executables
 - Build hyperledger SDK packages
 - Build hyperledger container images
 - Create  
 - ...
 
## Status

In development

## Requirements

- [Install Ansible](http://docs.ansible.com/ansible/intro_installation.html)
- [Ubuntu 16.04 machines] (https://cloud-images.ubuntu.com/releases/16.04/)
- Clone this project into a directory.

If you will be using an Ubuntu system as Ansible controller, then you can
easily setup an environment by running the following script. If you have
other system as your Ansible controller, you can do similar steps to setup
the environment, the command may not be exact the same but the steps you
need to do should be identical.

    sudo apt-get update

    sudo apt-get install python-dev python-pip libssl-dev libffi-dev -y
    sudo pip install --upgrade pip

    sudo pip install six==1.10.0
    sudo pip install ansible==2.2.1.0

    git clone https://github.com/litong01/fabric-deploy.git

This project requires that you use Ansible version 2.2.1.0 or above


## Run the script to deploy hyperledger fabric

With your cloud environment set, you should be able to run the script::

    ansible-playbook -e "action=apply env=os password=XXXXX" devbuild.yml

The above command will stand up a hyperledger cluster at the environment
defined in vars/ubuntu.yml file. Replace xxxxx with your own password.


## The results of the work load successful run

If everything goes well, it will accomplish the following::

    1. Provision 3 nodes or the number of nodes configured by stack_size
    2. Create security group
    3. Add security rules to allow ping, ssh, and kubernetes ports
    4. Install common software onto each node such as docker
    5. ...

## The method for running just a play, not the entire playbook

The script will create an ansible inventory file named runhosts at the very
first time you run the playbook, the inventory file will be place at a
directory named "run" at the root directory of the playbook. This file will be
updated in later runs if there are changes such as adding or removing hosts.
With this file, if you like to run only few plays, you will be able to do
that by following the example below:

    ansible-playbook -i run/runhosts -e "action=apply env=coreos password=XXXXX" devuild.yml
    --tags "common,master"

The above command will use the runhosts inventory file and only run plays
named common and master, all other plays in the play book will be skipped. All
available plays can be found in the site.yml file.


## Next Steps

### Check that everything is running correctly

If there are no errors, you can use kubectl to work with your kubernetes
cluster. Kubernetes and cockroachdb dashboards should be available at
different ports. Point your browser to an accessible IP address with the
following ports and your browser should show the dashboards.


## Cleanup

Once you're done with it, don't forget to nuke the whole thing::

    ansible-playbook -e "action=destroy env=coreos password=XXXXX" devbuild.yml

The above command will destroy all the resources created.
