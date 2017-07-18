Provision VMs against AWS cloud

Ansible uses Boto for AWS interactions, so you'll need that installed on your ansible controller. In certain casesm we may need  AWS CLI tools, so good to have to those rpms installed as well

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
