# Ansible Dynamic Inventory with AWS EC2 Plugin

With rapidly scaling cloud environment, its difficult to maintain couple of things due to scaling operatios being done automaically based on load and other parameters. Here we are going to foucs mainly on how to use Ansible to create a dynamic inventory using AWS EC2 plugin. I belive you used ansible for your daily operations and have some knowledge on ansible.

Ansible dynamic inventory plugin is supported from Ansible 2.7 version. Make sure you use ansible 2.7+ version or upgrade your ansible.

### Install ansible (if not installed already)
```
sudo apt-get update -y
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update -y
sudo apt-get install ansible -y
sudo ansible --version
```
### Get ansible config path from ansible version command
```
ansible 2.7.1
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.17 (default, Apr 15 2020, 17:20:14) [GCC 7.5.0]
```
### Add the following in ansible config file (Only needed if plugin is not enabled by default)
```
[inventory]
enable_plugins = aws_ec2, host_list, script, auto, yaml, ini, toml
```
### Create aws_dyinv.yml file
```
plugin: aws_ec2
boto_profile: default
regions:
  - us-east-2
filters:    
    instance-state-name: running
    tag:Env: Prod

keyed_groups:
  - key: tags.Role
    separator: ""

compose:
  # set the ansible_host variable to connect with the private IP address without changing the hostname
  ansible_host: private_ip_address
```

### Sample Output of dynamic inventory
```
root@ip-172-31-36-29:/home/ubuntu# ansible-inventory -i aws_dyinv.yml --graph
@all:
  |--@Backend_App:
  |  |--ec2-18-216-193-29.us-east-2.compute.amazonaws.com
  |  |--ec2-18-219-20-112.us-east-2.compute.amazonaws.com
  |  |--ec2-18-224-182-42.us-east-2.compute.amazonaws.com
  |--@Frontend_App:
  |  |--ec2-18-191-224-147.us-east-2.compute.amazonaws.com
  |  |--ec2-18-219-113-21.us-east-2.compute.amazonaws.com
  |--@aws_ec2:
  |  |--ec2-18-191-224-147.us-east-2.compute.amazonaws.com
  |  |--ec2-18-216-193-29.us-east-2.compute.amazonaws.com
  |  |--ec2-18-219-113-21.us-east-2.compute.amazonaws.com
  |  |--ec2-18-219-20-112.us-east-2.compute.amazonaws.com
  |  |--ec2-18-224-182-42.us-east-2.compute.amazonaws.com
  |--@ungrouped:
```
Let's see about the above file aws_dyinv.yml in detail. 
