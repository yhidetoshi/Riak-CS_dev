![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Riak-CS/basho-icon.jpeg)
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Riak-CS/riak-cs-image.png)

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Riak-CS/riak-cs-fig1.png)

**[環境]**
- Vagrant
  - CentOS6.7
    - node1:192.168.33.10
    - node2:192.168.33.11
    - node3:192.168.33.12
  - Riak-CS 3台リング構成


```
# wget http://s3.amazonaws.com/downloads.basho.com/riak/1.4/1.4.1/rhel/6/riak-1.4.1-1.el6.x86_64.rpm
# wget http://s3.amazonaws.com/downloads.basho.com/stanchion/1.4/1.4.0/rhel/6/stanchion-1.4.0-1.el6.x86_64.rpm
# wget http://s3.amazonaws.com/downloads.basho.com/riak-cs/1.4/1.4.0/rhel/6/riak-cs-1.4.0-1.el6.x86_64.rpm
```


**[node1]**
- 必要なモジュールをダウンロードする
  - riak
  - riak-cs
  - stanchion

```
[node1]
# wget http://s3.amazonaws.com/downloads.basho.com/riak/1.4/1.4.1/rhel/6/riak-1.4.1-1.el6.x86_64.rpm
# wget http://s3.amazonaws.com/downloads.basho.com/stanchion/1.4/1.4.0/rhel/6/stanchion-1.4.0-1.el6.x86_64.rpm
# wget http://s3.amazonaws.com/downloads.basho.com/riak-cs/1.4/1.4.0/rhel/6/riak-cs-1.4.0-1.el6.x86_64.rpm

# rpm -ivh riak-1.4.1-1.el6.x86_64.rpm
# rpm -ivh riak-cs-1.4.0-1.el6.x86_64.rpm
# rpm -ivh stanchion-1.4.0-1.el6.x86_64.rpm

# cd /etc/riak
# cp -pr app.config app.config.org
```
- アドレスを設定する
```
# diff -u app.config.org app.config
--- app.config.org	2013-08-01 21:29:20.000000000 +0000
+++ app.config	2016-07-23 12:21:24.037029730 +0000
@@ -12,7 +12,7 @@

             %% pb is a list of IP addresses and TCP ports that the Riak
             %% Protocol Buffers interface will bind.
-            {pb, [ {"127.0.0.1", 8087 } ]}
+            {pb, [ {"192.168.33.10", 8087 } ]}
             ]},

  %% Riak Core config

```
