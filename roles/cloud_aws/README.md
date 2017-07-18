Provision VMs against AWS cloud

Ansible uses Boto for AWS interactions, so you'll need that installed on your control host. We're also going to make some use of the AWS CLI tools, so get those too. Your platform may differ, but the following will work for most platforms:

https://atplanet.co/blog/ec2-auto-scaling-with-ansible.html
---------
Pre-reqs 
---------

Must:
-----
sudo pip install boto==2.47.0

Nice to have:
-------------
sudo apt-get install awscli -y
sudo apt-get install ec2-api-tools -y
