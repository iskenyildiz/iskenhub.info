---
layout: post
title:  "Configuring HAProxy to Load Balance MYSQL Servers on Ubuntu 18.04"
date:   2019-09-19 11:31:21 +0300
categories: jekyll update
---


We did load balancing for HTTP. What if we need to load balance a cluster of databases? We would need a way to be able to direct the requests for each database.

For this we will be using two MYSQL Servers and one HAPRoxy server to load balance them. 

## MYSQL Server 1: 192.168.56.102
## MYSQL Server 2: 192.168.56.103
## HAProxy Server: 192.168.56.101

We can do the Keepalived configuration too but we did that in the previous blog so we won't be doing it here. But know that there is no problem in adding another HAproxy server and using keepalived.

First install mysql server on the machines 1 and 2.

`
sudo apt install mysql-server
`

We want our HAProxy server to be able to check the status of the databases but this isn't possible until we declare a user with an IP address since mysql works locally for root users.

The other user 'haproxy_root' is used to access the sql privileges for queries since we will be needing it to look for load balancing.

` sudo mysql -u root -p`

inside:

```
INSERT INTO mysql.user (Host,User,ssl_cipher,x509_issuer,x509_subject) values ('192.168.56.101','haproxy_checker','','','');

FLUSH PRIVILEGES;
```
I've added the ciphers because it was needed for me to be able to create a user. You can disable ssl option on the mysql config files if you do not want to add the cipher.

```
GRANT ALL PRIVILEGES ON *.* TO 'haproxy_root'@'192.168.56.101' IDENTIFIED BY 'password' WITH GRANT OPTION; 

FLUSH PRIVILEGES;
```

Next we want to give server IDs to the machines since they might be declared as '0'.

Look at the server IDs for both machines using `show variables like 'server_id';`

If you see different IDs on servers that is fine but if you still want to change them use:

`SET GLOBAL server_id=your_id`

Now we want to install MYSQL Client on the HAProxy server for monitoring the mysql servers.

` sudo apt install mysql-client`

We can query databases with our 'haproxy_user' that we declared previously 

` mysql -h 10.0.0.1 -u haproxy_root -p -e "SHOW DATABASES"`

If you cannot reach the server you might need to go and edit

` sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf` 

and comment out ` bind=127.0.0.1`

After that go to the haproxy config file

` nano /etc/haproxy/haproxy.cfg `


By default HAProxy doesn't store the log files inside the /var/log/ directory. We need to make sure this isn't the case.

add the following under the global section.

`
    log 127.0.0.1 local0 notice
`

Next add the listen blocks.

```

listen mysql-cluster
    bind 127.0.0.1:3306
    mode tcp
    option mysql-check user haproxy_checker
    balance roundrobin
    server mysql1 192.168.56.102:3306 check
    server mysql2 192.168.56.103:3306 check # add 'weight 2' if you want the load balancer to redirect two times before going back to server1.
listen 0.0.0.0:8080
    mode http
    stats enable
    stats uri /
    stats auth Username:YourPassword # change username and password for yourself
    
```
You can go to http://192.168.56.101:8080 to see the statistics page which you enabled tracking by declaring the user 'haproxy_checker'.

You can either see if load balancing works by looking at databases if you created on either of the machines.

`
mysql -h 127.0.0.1 -u haproxy_root -p -e "SHOW DATABASES"
`

or you can do so by looking at 'server_id'.

`
mysql -h 127.0.0.1 -u haproxy_root -p -e "show variables like 'server_id'"
`

[![Screenshot-from-2019-09-19-11-09-26.png](https://i.postimg.cc/hj21hT5y/Screenshot-from-2019-09-19-11-09-26.png)](https://postimg.cc/zbRgcb5K)

do this many times and you will notice the change of the server IDs.

Now if you stop a mysql server and head into `tail -f /var/log/haproxy.log` you will see a message indicating:

```
Sep 19 11:17:50 localhost haproxy[13041]: Server mysql-cluster/mysql1 is DOWN, reason: Layer4 connection problem, info: "Connection refused", check duration: 0ms. 1 active and 0 backup servers left. 0 sessions active, 0 requeued, 0 remaining in queue.
```

Means you can monitor the availability of the servers.















