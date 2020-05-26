# Sensu Go Assets

Assets are shareable, reusable packages that make it easier to deploy Sensu plugins. Instead of configuring the environment on all the nodes seperately, Assets will hold all the packages you need to run on the node and when the check is executed, the packages and checks will be downloaded to the respective nodes. In this way you don't need to configure everything on the individual nodes, all the necessary packages/code/script will be pushed from a central place to your infrastructure/nodes.

Sensu provides 2 ways of configuring assets.
1. Using Bonsai
2. Configure Locally

Let's see how we can configure a cpu-check and see how assets are useful.

# Configure Asset using Bonsai
Bonsai, the Sensu asset index, is a centralized place for downloading and sharing plugin assets.

### Search for cpu-check plugin
Go to [Bonsai](https://bonsai.sensu.io/assets/) and search for cpu. You will find [this](https://bonsai.sensu.io/assets/sensu-plugins/sensu-plugins-cpu-checks) plugin. When you see the overview of the asset plugin in bin directory it has couple of scripts, we are going to use check-cpu.rb here.

### Add the asset to your sensu-backend server
```
sensuctl asset add sensu-plugins/sensu-plugins-cpu-checks
```
### Check for the asset
```
root@ip-172-31-38-43:/home/ubuntu# sensuctl asset list
                   Name                                                              URL                                               Hash    
 ──────────────────────────────────────── ────────────────────────────────────────────────────────────────────────────────────────── ─────────  
  sensu-plugins/sensu-plugins-cpu-checks   //assets.bonsai.sensu.io/.../sensu-plugins-cpu-checks_4.1.0_debian9_linux_amd64.tar.gz     9dd717a  
  sensu-plugins/sensu-plugins-cpu-checks   //assets.bonsai.sensu.io/.../sensu-plugins-cpu-checks_4.1.0_debian_linux_amd64.tar.gz      f0435fd  
  sensu-plugins/sensu-plugins-cpu-checks   //assets.bonsai.sensu.io/.../sensu-plugins-cpu-checks_4.1.0_centos7_linux_amd64.tar.gz     8a01862  
  sensu-plugins/sensu-plugins-cpu-checks   //assets.bonsai.sensu.io/.../sensu-plugins-cpu-checks_4.1.0_centos6_linux_amd64.tar.gz     f42be79  
  sensu-plugins/sensu-plugins-cpu-checks   //assets.bonsai.sensu.io/.../sensu-plugins-cpu-checks_4.1.0_alpine3.8_linux_amd64.tar.gz   7a5ad2d  
  sensu-plugins/sensu-plugins-cpu-checks   //assets.bonsai.sensu.io/.../sensu-plugins-cpu-checks_4.1.0_alpine_linux_amd64.tar.gz      a67676f  
```
As you see here, asset sensu-plugins/sensu-plugins-cpu-checks is added and the URL it is pointing is to bonsai.

### Ruby runtime plugin
Ruby runtime asset should allow Ruby-based scripts (e.g. Sensu Community plugins) to be packaged as separate assets containing Ruby scripts and any corresponding gem dependencies. In this way, a single shared Ruby runtime may be delivered to systems running the new Sensu Go Agent via the new Sensu's new Asset framework (i.e. avoiding solutions that would require a Ruby runtime to be redundantly packaged with every ruby-based plugin).

### Add the asset to your sensu-backend server
```
sensuctl asset add sensu/sensu-ruby-runtime
```

So far we have created 2 asset, sensu-plugins-cpu-checks which has checks to execute and sensu-ruby-runtime which has ruby scripts and any corresponding gem dependencies.

Let's create a check now and see how the cpu-check works.
