# Install and configure Sensu go on Ubuntu 18.04

The installation is pretty straightforward. Sensu go removed rabbitmq, redis and replaced with inbuilt transport and state storage layer.

### Install sensu backend
```
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.deb.sh | sudo bash
sudo apt-get install sensu-go-backend
```
### Pull basic config for sensu backend
```
sudo curl -L https://docs.sensu.io/sensu-go/latest/files/backend.yml -o /etc/sensu/backend.yml
```
### Update the sensu backend config (/etc/sensu/backend.yml)
```
---
state-dir: "/var/lib/sensu/sensu-backend" #etcd datastorage instead of redis
cache-dir: "/var/cache/sensu/sensu-backend"
config-file: "/etc/sensu/backend.yml"
debug: false
agent-port: 8081
dashboard-port: 3000
```
Make sure you update the above values in the configuration.

### Start sensu-backend
```
sudo service sensu-backend start
```
### Create admin user and password
```
export SENSU_BACKEND_CLUSTER_ADMIN_USERNAME=admin
export SENSU_BACKEND_CLUSTER_ADMIN_PASSWORD=password
sensu-backend init
```
You will now be able to see the sensu dashboard on http://your-public-ip:3000


I will be installing sensuctl (the command line utils) on a seperate node just to avoid unnecessary logins to the sensu-backend server.

### Install sensuctl
```
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.deb.sh | sudo bash
sudo apt-get install sensu-go-cli
```
### Configure sensuctl with sensu-backend
```
sensuctl configure -n \
--username 'admin' \
--password 'password' \
--namespace default \
--url 'http://your-sensu-backend-ip:8080'
```

### Install sensu agents
```
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.deb.sh | sudo bash
sudo apt-get install sensu-go-agent
```
### Pull sensu-agent configuration
```
sudo curl -L https://docs.sensu.io/sensu-go/latest/files/agent.yml -o /etc/sensu/agent.yml
```
### Configure sensu-agent (/etc/sensu/agent.yml)
```
---
name: "consul-server-1"
namespace: "default"
subscriptions:
  - system
labels:
  server_region: "us-east-2"
  role: "consul-agent"
annotations:
  tenancy: "shared"
backend-url:
  - "ws://172.31.38.43:8081" #Sensu-backend IP address
cache-dir: "/var/cache/sensu/sensu-agent"
config-file: "/etc/sensu/agent.yml"
```
We will see the details of the configuration in upcoming tutorials.

### Start sensu-agent
```
service sensu-agent start
```

Go to the Sensu go UI and now you can see your server on the sensu go dashboard. You can configure as many servers as you want.
