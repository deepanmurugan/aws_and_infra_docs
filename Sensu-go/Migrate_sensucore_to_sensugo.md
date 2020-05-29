# Migrate Sensu Core to Sensu Go

You can do an inservice migration or migration with a different host. In this tutorial, I am not disturbing the old sensu-core setup and install sensu components on new server and migrate the old sensu-client to new sensu-go-agent and eventually remove the sensu-client package.

Migration strategy:

1. Create a new server and install sensu-backend and sensuctl.
2. Install sensu-agent on the old sensu-client node.
3. Update the sensu-agent configuration to point to sensu-backend server.
4. Install sensu translator and translate old sensu-checks to new ones.
5. Create sensu checks in new sensu-go server.
6. Add file /etc/default/sensu-agent to include new sensu-plugin path.
```
PATH=/opt/sensu-plugins-ruby/embedded/bin:/opt/sensu/embedded/bin:/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
```
7. Add file /etc/default/sensu-backend to include new sensu-plugin path.
```
PATH=/opt/sensu-plugins-ruby/embedded/bin:/opt/sensu/embedded/bin:/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
```
8. Save list of installed plugins on target hosts.
```
/opt/sensu/embedded/bin/gem list --local --no-details --no-versions |grep sensu-plugins
```
9. Install sensu-plugins-ruby package
```
sudo apt install sensu-plugins-ruby
```
10. Create symlink for ruby and gem packages
```
ln -s /opt/sensu-plugins-ruby/embedded/bin/ruby /usr/bin/ruby
ln -s /opt/sensu-plugins-ruby/embedded/bin/gem /usr/bin/gem
```
11. Sensu-install or /opt/sensu-plugins-ruby/embedded/bin/gem to install sensu-plugins
```
gem install sensu-plugins (or)
sensu-install -p sensu-plugins
```
12. Remove old sensu-client packages whenever your new checks are passing.
```
sudo apt purge sensu-client
```
