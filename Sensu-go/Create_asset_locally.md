# Create Asset Locally

If you don't want to use Bonsai to download the asset packages since it is public and anyone can contribute to it, you can download all the packages and host a webserver, use the packages from that webserver to create asset, so that your asset will have more control and its in your local network always.

Here I am using a seperate node just to keep all the packages and run a webserver in order to better control the packages. I am using apache as a webserver to host the packages.

### Create a webserver
```
sudo apt install apache2
```
### Download and keep the asset packages
```
As before, search for memory in bonsai and find the [relevant package](https://assets.bonsai.sensu.io/db2bd124b2bada73408df8c453f4fb676775b306/sensu-plugins-memory-checks_4.1.1_debian_linux_amd64.tar.gz). Go to release assets tab to see all version packages. Download the relevant package and I am downloading debian as I am using Ubuntu.

cd /var/www/html
wget https://assets.bonsai.sensu.io/db2bd124b2bada73408df8c453f4fb676775b306/sensu-plugins-memory-checks_4.1.1_debian_linux_amd64.tar.gz
wget https://assets.bonsai.sensu.io/5123017d3dadf2067fa90fc28275b92e9b586885/sensu-ruby-runtime_0.0.10_ruby-2.4.4_debian_linux_amd64.tar.gz
```
