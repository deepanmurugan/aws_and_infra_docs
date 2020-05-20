# Install and configure Redis and Sentinel as HA and configure auto update of Route53 DNS

A little background about redis, Redis (Remote Dictionary Server) is a very popular and widely-used open source, fast, distributed and efficient in-memory key-value database/data structure server.

It offers a rich set of features that make it effective for a wide range of use cases: as a database, caching layer, message broker, or queue; applicable in web applications, chat and messaging applications, gaming, real-time data analytics and so much more.

It supports flexible data structures, master-slave asynchronous replication to scale read performance and guard against data loss, client-side sharding to scale write performance, two forms of persistence for writing in-memory data to disk in a compact format, clustering, and partitioning. It also features automatic failovers for high availability deployment via Redis Sentinel, Lua scripting, transactions, and many more.

In this tutorial we are going to install and configure Redis on the below nodes. With this set up we have 1 read/write primary master node and 2 read only slave nodes.
172.31.43.100 - Redis server 1 - Sentinel 1 - Master - DNS ()
172.31.36.29 - Redis server 2 - Sentinel 2 - Slave
172.31.33.12 - Redis server 3 - Sentinel 3 - Slave

HA considerations:
*Install 2 nodes in a region with 2 different availability zones
*Install 3rd node in other region

Let's see how we can install and configure redis and sentinel with auto update of AWS Route53 DNS record. First let's configure redis and then we will go for sentinel.

### Install Redis Server and Redis Sentinel on all 3 nodes
```
sudo apt install redis-server redis-sentinel -y
```
### Configure Redis on Master node (Make sure you edit these values in the existing config files)
```
root@ip-172-31-36-29:/etc/redis#
bind 127.0.0.1 172.31.36.29 #Your IP address where redis have to bind
protected-mode no #allow communication with the replicas
port 6379 #port where redis runs
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
supervised systemd
pidfile "/var/run/redis/redis-server.pid"
loglevel notice
logfile "/var/log/redis/redis-server.log"
databases 16
always-show-logo yes
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename "dump.rdb"
dir "/var/lib/redis"
masterauth "redispass"
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
requirepass "redispass" #This will force redis to use a password
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
slave-lazy-flush no
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble no
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
```
### Restart Redis service
```
sudo service redis restart
```
### Configure Redis Slave nodes
```
bind 127.0.0.1 172.31.43.100
protected-mode no
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
supervised systemd
pidfile "/var/run/redis/redis-server.pid"
loglevel notice
logfile "/var/log/redis/redis-server.log"
databases 16
always-show-logo yes
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename "dump.rdb"
dir "/var/lib/redis"
masterauth "redispass" #Master redis password to make connection for replication
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
requirepass "redispass"
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
slave-lazy-flush no
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble no
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
slaveof 172.31.36.29 6379 #Slave of parameter defines this server is a slave of master host 172.31.36.29
```
### Restart Redis service on slaves
```
sudo service redis restart
```
### Check if nodes are able to communicate properly
```
root@ip-172-31-36-29:/etc/redis# redis-cli
127.0.0.1:6379> auth redispass
OK
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=172.31.43.100,port=6379,state=online,offset=21478288,lag=0
slave1:ip=172.31.33.12,port=6379,state=online,offset=21478274,lag=0
master_replid:430086cc7038ecb6205a769db062c5b9ab7d1985
master_replid2:f75e736d4f15df807c851af2332936b5f31d08fa
master_repl_offset:21478288
second_repl_offset:5470768
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:20429713
repl_backlog_histlen:1048576
```
Here when we execute the `info replication` on master node, we can see that there are 2 slaves connected to master already and they are in sync. You can set some key value pair in master and check if it's replicated in slave nodes.

### Configure Sentinel on Master node
```
daemonize yes
pidfile "/var/run/sentinel/redis-sentinel.pid"
logfile "/var/log/redis/redis-sentinel.log"
sentinel myid 22f7227a89fd10244d2073f8c358bce2f1e3c563 #Unique ID for each sentinel node
sentinel monitor mymaster 172.31.36.29 6379 2 #Master node IP
protected-mode no
port 26379 #Port where sentinel runs
dir "/var/lib/redis"
sentinel down-after-milliseconds mymaster 3000 #3000 milliseconds (3secs) to declare if the master is down when there is no communication with master
sentinel auth-pass mymaster redispass #Your redis password
```
### Restart Redis Sentinel on Master
```
sudo service redis-sentinel restart
```
### Configure Sentinel on 2 Slave nodes
```
daemonize yes
pidfile "/var/run/sentinel/redis-sentinel.pid"
logfile "/var/log/redis/redis-sentinel.log"
protected-mode no
port 26379
dir "/var/lib/redis"
sentinel myid a451b26ead8036d4fb6d2e789b57550b9ff1deed
sentinel monitor mymaster 172.31.36.29 6379 2
sentinel down-after-milliseconds mymaster 3000
sentinel auth-pass mymaster redispass
```
### Restart Redis Sentinel on Slaves
```
sudo service redis-sentinel restart
```
### Configure Route53 DNS Entry for Master Node
I have created an internal domain in AWS Route53 learnes.internal.com. Let's create an `A record` in the domain as master.redis.learnes.internal.com with A record value as master redis IP `172.31.36.29`

When any application wants to write to Redis server it has to write the master redis server. Whenever any redis server failover happens, your application will fail as well and it requires manual intervention to change the redis master server in the application configuration or in the DNS A record value. So we are using a script to automatically detect the redis master change/failover and update the DNS record automatically.

```
root@ip-172-31-43-100:/etc/redis# dig +short master.redis.learnes.internal.com
172.31.36.29
```

I am going to use supervisor process to run the DNS failover script for every 10sec.

### Install Supervisor on all 3 nodes
```
sudo apt install supervisor -y
```
### Supervisor config file on all 3 nodes
```
root@ip-172-31-36-29:/etc/supervisor# cat supervisord.conf 
; supervisor config file

[unix_http_server]
file=/var/run/supervisor.sock   ; (the path to the socket file)
chmod=0700                       ; sockef file mode (default 0700)

[supervisord]
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = /etc/supervisor/conf.d/*.conf
```
### DNS Update Script for Redis Master server on all 3 nodes
```
#!/bin/bash

while true
do

dns_ip=`dig +short master.redis.learnes.internal.com`
current_ip=`hostname -I`
master_role="master"
hosted_zone="Z03711351P6IX6D1O7D"

#Check if this node is redis master
redis-cli -p 6379 -a redispass info replication| grep role | cut -d ":" -f 2 | grep master
redis_role=`echo $?`

#Check if DNS A record value is not as same as local IP. If not same then update DNS A record
if [ $dns_ip != $current_ip ] && [ $redis_role -eq 0 ]
then
echo "Redis failover happened"
aws route53 change-resource-record-sets --hosted-zone-id `echo $hosted_zone` --change-batch "file:///home/ubuntu/update_dns.json"
elif [ $dns_ip == $current_ip ] && [ $redis_role -eq 0 ]
then
echo "No redis failover happened"
else
echo "This is slave node..."
fi

sleep 10
done
```
The above script should be run on all 3 nodes. This checks if the localhost is the redis master server. If so, then it will check if the DNS is pointing to different host, if not then NO FAILOVER or it will consider it as FAILOVER and update the DNS A record with the localhost.

### Configure this script as supervisord process on all 3 nodes
```
root@ip-172-31-36-29:/etc/supervisor/conf.d# cat redis_dns.conf 
;------------------------------------------------------------------
; List of all the applications/daemons to be run on the this server
;------------------------------------------------------------------
;
; Redis Agent
[program:redis_dns_update]
command = /home/ubuntu/script.sh
stdout_logfile = /home/ubuntu/redis_supervisor.log
redirect_stderr = true
;-- Total log size of 200M
stdout_logfile_maxbytes = 20MB
stdout_logfile_backups = 10
autostart = true
autorestart = true
startsecs = 5
startretries = 3
```
### Restart supervisor service
```
sudo service supervisor restart
```
### Start redis_dns_update process if its not already running
```
root@ip-172-31-36-29:/etc/supervisor/conf.d# sudo supervisorctl
redis_dns_update                 RUNNING   pid 4055, uptime 22:00:01
```
### To start the service
```
sudo supervisorctl start redis_dns_update
```

### Testing failover
```
Login to master redis node and stop redis for few seconds

root@ip-172-31-36-29:/etc/supervisor/conf.d# dig +short master.redis.learnes.internal.com
172.31.36.29

root@ip-172-31-36-29:/etc/supervisor/conf.d# redis-cli
127.0.0.1:6379> auth redispass
OK
127.0.0.1:6379> debug sleep 100

On slave node: (This becomes the new master and already the stopped slave is joined with this node as slave)

root@ip-172-31-33-12:/home/ubuntu# redis-cli
127.0.0.1:6379> auth redispass
OK
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=172.31.43.100,port=6379,state=online,offset=21918983,lag=0
slave1:ip=172.31.36.29,port=6379,state=online,offset=21919125,lag=0
master_replid:3c5b77b8a69dcba3a61e2023e1f2063b70f77c4a
master_replid2:430086cc7038ecb6205a769db062c5b9ab7d1985
master_repl_offset:21919125
second_repl_offset:21892073
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:20870550
repl_backlog_histlen:1048576

root@ip-172-31-33-12:/home/ubuntu# dig +short master.redis.learnes.internal.com
172.31.33.12

We can see now the script triggered and DNS A record now updated automatically to redis master server.

Logs of new master node:
master
Redis failover happened
{
    "ChangeInfo": {
        "Id": "/change/C09218263UOP7SA5UKY6O",
        "Status": "PENDING",
        "SubmittedAt": "2020-05-20T15:02:54.323Z",
        "Comment": "optional comment about the changes in this change batch request"
    }
}
master
No redis failover happened
master
No redis failover happened

Logs of slave node:

No redis failover happened
master
No redis failover happened
master
This is slave node...
This is slave node...
This is slave node...
This is slave node...
```
Once this config is done, you don't need to worry about redis failover any more. Whenever master goes down, it will automatically update DNS entry and when you point application to DNS entry (master.redis.learnes.internal.com), you don't need to worry about any failover.
