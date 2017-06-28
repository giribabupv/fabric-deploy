# Hyperledger Fabric Deployment

This ansible playbook project will accomplish the following tasks

 - Provision virtual machine nodes to participate in fabric network
 - Install necessary hyperledger dependent libraries and packages
 - Setup overlay network so containers can communicate cross multiple docker host
 - Install registrator and dns services so that containers can be referenced by name
 - Build hyperledger fabric artifacts
 - Run hyperledger fabric tests
 - Generate fabric network certificats, genesis blocks, transaction file etc
 - Push new or tagged fabric images onto all docker hosts
 - Deploy fabric network
 
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
    sudo pip install ansible==2.3.0.0
    git clone https://github.com/litong01/fabric-deploy.git

This project requires that you use Ansible version 2.3.0.0 or above


## Run the script to provision docker hosts and install dependent libraries.

With your cloud environment set, run the script to provision docker hosts::

    ansible-playbook -e "mode=apply env=os password=XXXXX" devenv.yml


The above command will stand up a hyperledger cluster at the environment
defined in vars/os.yml file. Replace xxxxx with your own password from your
cloud provider.


## Setup the overlay network for the docker hosts created above::

    ansible-playbook -i run/runhosts -e "mode=apply env=os" overlay.yml

## Setup the fabric network::

    ansible-playbook -i run/runhosts -e "mode=apply env=bc1st" fabricbuild.yml

The env value in the command indicates which fabric network configuration to use.
In above example, ansible looks for a file in vars directory named bc1st.yml,
you can create as many files in that directory to reflect your own network. Here
is the bc1st.yml (short for block chain 1st network)::

	    ---
	    # The url to the fabric source repository
	    GIT_URL: "http://gerrit.hyperledger.org/r/fabric"

	    # The gerrit patch set reference, should be automatically set by gerrit
	    GERRIT_REFSPEC: "refs/tags/v1.0.0-rc1"

	    # This variable defines fabric network attributes
	    fabric: {
  	      ssh_user: "ubuntu",
	      network: {
            fabric001: {
              cas: ["ca.orga", "ca.orgb"],
      	      peers: ["leader@1stpeer.orga", "leader@1stpeer.orgb"],
              orderers: ["1storderer.orgc", "1storderer.orgd"],
              zookeepers: ["zookeeper1st"],
      	      kafkas: ["kafka1st"]
    	    },
    	    fabric002: {
      	      cas: ["ca.orgc", "ca.orgd"],
              peers: ["anchor@2ndpeer.orga", "anchor@2ndpeer.orgb"],
              orderers: ["2ndorderer.orgc", "2ndorderer.orgd"],
              zookeepers: ["zookeeper2nd"],
              kafkas: ["kafka2nd"]    
            },
            fabric003: {
              peers: ["worker@3rdpeer.orga", "worker@3rdpeer.orgb"],
              zookeepers: ["zookeeper3rd"],
              kafkas: ["kafka3rd", "kafka4th"]    
            }
          },
          baseimage_tag: "x86_64-1.0.0-rc1"
        }


## The method for running just a play, not the entire playbook

The script will create an ansible inventory file named runhosts at the very
first time you run the playbook, the inventory file will be place at a
directory named "run" at the root directory of the playbook. This file will be
updated in later runs if there are changes such as adding or removing hosts.
With this file, if you like to run only few plays, you will be able to do
that by following the example below:

    ansible-playbook -i run/runhosts -e "mode=apply env=os password=XXXXX" devenv.yml
    --tags "postprovision,fastinitnode"

The above command will use the runhosts inventory file and only run play
named postprovision and fastinitnode, all other plays in the play books will
be skipped. All available plays can be found in the site.yml file.


## Next Steps

### Check that everything is running correctly

If there are no errors, you can use kubectl to work with your kubernetes
cluster. Kubernetes and cockroachdb dashboards should be available at
different ports. Point your browser to an accessible IP address with the
following ports and your browser should show the dashboards.


## Cleanup

Once you're done with it, don't forget to nuke the whole thing::

    ansible-playbook -e "mode=destroy env=os password=XXXXX" devenv.yml

The above command will destroy all the resources created.
