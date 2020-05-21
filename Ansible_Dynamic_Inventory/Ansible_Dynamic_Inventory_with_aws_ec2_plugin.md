# Ansible Dynamic Inventory with AWS EC2 Plugin

With rapidly scaling cloud environment, its difficult to maintain couple of things due to scaling operations being done automaically based on load and other parameters. You might have seen your autoscaling launched few more EC2 instances and when you use ansible (static inventory) you might miss those new instances. So here we are going to foucs mainly on how to use Ansible to create a dynamic inventory using AWS EC2 plugin. I belive you used ansible for your daily operations and have some knowledge on ansible.

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

plugin: aws_ec2 - Defines which plugin to use
boto_profile: default - My boto profile on this machine where I run ansible inventory command
regions - AWS EC2 regions, you can mention any number of regions
filters - Used to filter out EC2 instances based on conditions. Here we are selecting only running instances with tag `Env: Prod`
keyed_groups - Define how you want to create groups. Here we are again using tag called `Role` for creating groups
compose - Output (you can mention private_ip_address or public_ip_address based on your need)

### Run ansible playbook

Suppose if I want to run some ansible playbook for all the servers in Backend_App, you can use the command like below. Below is the sample playbook that I trigger.

```
ubuntu@ip-172-31-36-29:~$ cat play.yml 
- name : Playbook to create a folder 
  hosts: "{{host}}"
  user: ubuntu 
  become: yes
  gather_facts: True

  tasks:
    - name: Create a folder 
      file:
        path: /home/ubuntu/folder
        state: directory
        owner: ubuntu
        mode: 0755
```
```
ubuntu@ip-172-31-36-29:~$ ansible-playbook --private-key ohio.pem -i aws_dyinv.yml --extra-vars "host=Backend_App ansible_python_interpreter=/usr/bin/python3" play.yml

PLAY [Playbook to create a folder] ******************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [ec2-18-219-20-112.us-east-2.compute.amazonaws.com]
ok: [ec2-18-216-193-29.us-east-2.compute.amazonaws.com]
ok: [ec2-18-224-182-42.us-east-2.compute.amazonaws.com]

TASK [Create a folder] ******************************************************************************************************************
ok: [ec2-18-219-20-112.us-east-2.compute.amazonaws.com]
ok: [ec2-18-216-193-29.us-east-2.compute.amazonaws.com]
ok: [ec2-18-224-182-42.us-east-2.compute.amazonaws.com]

PLAY RECAP ******************************************************************************************************************************
ec2-18-216-193-29.us-east-2.compute.amazonaws.com : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ec2-18-219-20-112.us-east-2.compute.amazonaws.com : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ec2-18-224-182-42.us-east-2.compute.amazonaws.com : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
While triggering my playbook I selected Backend_App group and it ran for those hosts only. Also I'm forcing ansible to use python3 using this variable ansible_python_interpreter=/usr/bin/python3 but its optional.

You can also use -l option as below and change `host: all` in your playbook.
```
ubuntu@ip-172-31-36-29:~$ ansible-playbook --private-key ohio.pem -i aws_dyinv.yml -l Backend_App play.yml

PLAY [Playbook to create a folder] ******************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [ec2-18-219-20-112.us-east-2.compute.amazonaws.com]
ok: [ec2-18-216-193-29.us-east-2.compute.amazonaws.com]
ok: [ec2-18-224-182-42.us-east-2.compute.amazonaws.com]

TASK [Create a folder] ******************************************************************************************************************
ok: [ec2-18-219-20-112.us-east-2.compute.amazonaws.com]
ok: [ec2-18-216-193-29.us-east-2.compute.amazonaws.com]
ok: [ec2-18-224-182-42.us-east-2.compute.amazonaws.com]

PLAY RECAP ******************************************************************************************************************************
ec2-18-216-193-29.us-east-2.compute.amazonaws.com : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ec2-18-219-20-112.us-east-2.compute.amazonaws.com : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ec2-18-224-182-42.us-east-2.compute.amazonaws.com : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
