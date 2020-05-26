# Create Asset Locally

If you don't want to use Bonsai to download the asset packages since it is public and anyone can contribute to it, you can download all the packages and host a webserver, use the packages from that webserver to create asset, so that your asset will have more control and its in your local network always.

Here I am using a seperate node just to keep all the packages and run a webserver in order to better control the packages. I am using apache as a webserver to host the packages.

### Create a webserver
```
sudo apt install apache2
```
### Download and keep the asset packages
As before, search for memory in bonsai and find the [relevant package](https://assets.bonsai.sensu.io/db2bd124b2bada73408df8c453f4fb676775b306/sensu-plugins-memory-checks_4.1.1_debian_linux_amd64.tar.gz). Go to release assets tab to see all version packages. Download the relevant package and I am downloading debian as I am using Ubuntu.
```
cd /var/www/html
wget https://assets.bonsai.sensu.io/db2bd124b2bada73408df8c453f4fb676775b306/sensu-plugins-memory-checks_4.1.1_debian_linux_amd64.tar.gz
wget https://assets.bonsai.sensu.io/5123017d3dadf2067fa90fc28275b92e9b586885/sensu-ruby-runtime_0.0.10_ruby-2.4.4_debian_linux_amd64.tar.gz
```
On sensu backend node, create a folder named assets and store all asset config files.
### Create asset definition file
```
mkdir -p /etc/sensu/assets
```
Create configuration file
```
root@ip-172-31-38-43:/etc/sensu/assets# cat local-sensu-memory-checks.yml 
---
type: Asset
api_version: core/v2
metadata:
  name: local/sensu-memory-checks
spec:
  url: http://172.31.40.205/sensu-plugins-memory-checks_4.1.1_debian_linux_amd64.tar.gz
  sha512: a1dff4bc3b890ee60bc559a64c9c89cf81f6638770c2c1bb50482050e95d70cc1b93f39ae5dd2903511f2dd685b1374bd4ddd2032fb0e28183bf5fb77a8065f2 
```
Where url is the place where I hosted the tar.gz asset packages and sha512sum is the shasum of file. name is the asset name when you create asset using this file.

### Create asset using sensuctl
```
sensuctl create -f /etc/sensu/assets/local-sensu-memory-checks.yml
```
Create one more asset for ruby runtime as well.
```
root@ip-172-31-38-43:/etc/sensu/assets# cat local-sensu-ruby-runtime.yml 
---
type: Asset
api_version: core/v2
metadata:
  name: local/sensu-ruby-runtime
spec:
  url: http://172.31.40.205/sensu-ruby-runtime_0.0.10_ruby-2.4.4_debian_linux_amd64.tar.gz
  sha512: a28952fd93fc63db1f8988c7bc40b0ad815eb9f35ef7317d6caf5d77ecfbfd824a9db54184400aa0c81c29b34cb48c7e8c6e3f17891aaf84cafa3c134266a61a
```
### Create asset using sensuctl
```
sensuctl create -f /etc/sensu/assets/local-sensu-ruby-runtime.yml
```
### List asset
```
                   Name                                                              URL                                               Hash    
 ──────────────────────────────────────── ────────────────────────────────────────────────────────────────────────────────────────── ───────── 
  local/sensu-memory-checks                //172.31.40.205/.../sensu-plugins-memory-checks_4.1.1_debian_linux_amd64.tar.gz            a1dff4b   
  local/sensu-ruby-runtime                 //172.31.40.205/.../sensu-ruby-runtime_0.0.10_ruby-2.4.4_debian_linux_amd64.tar.gz         a28952f  
```
### Configure check using this local asset packages
```
sensuctl check create local-memory-check \
--command 'check-memory.rb -w 60 -c 70' \
--interval 60 \
--subscriptions system \
--runtime-assets "local/sensu-memory-checks,local/sensu-ruby-runtime"
```



