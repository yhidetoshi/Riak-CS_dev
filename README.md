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

# cp -pr app.config app.config.org

```
- `/etc/riak/app.conf`を編集する
```
# diff -u app.config.org app.config
--- app.config.org	2013-08-01 21:29:20.000000000 +0000
+++ app.config	2016-07-23 12:40:00.676033801 +0000
@@ -12,14 +12,14 @@

             %% pb is a list of IP addresses and TCP ports that the Riak
             %% Protocol Buffers interface will bind.
-            {pb, [ {"127.0.0.1", 8087 } ]}
+            {pb, [ {"192.168.33.10", 8087 } ]}
             ]},

  %% Riak Core config
  {riak_core, [
               %% Default location of ringstate
               {ring_state_dir, "/var/lib/riak/ring"},
-
+	      {default_bucket_props, [{allow_mult, true}]},
               %% Default ring creation size.  Make sure it is a power of 2,
               %% e.g. 16, 32, 64, 128, 256, 512 etc
               %{ring_creation_size, 64},
@@ -80,8 +80,20 @@
  {riak_kv, [
             %% Storage_backend specifies the Erlang module defining the storage
             %% mechanism that will be used on this node.
-            {storage_backend, riak_kv_bitcask_backend},
-
+            %%{storage_backend, riak_kv_bitcask_backend},
+            {add_paths, ["/usr/lib64/riak-cs/lib/riak_cs-1.4.0/ebin"]},
+ 	    {storage_backend, riak_cs_kv_multi_backend},
+ 	    {multi_backend_prefix_list, [{<<"0b:">>, be_blocks}]},
+ 	    {multi_backend_default, be_default},
+ 	    {multi_backend, [
+   		{be_default, riak_kv_eleveldb_backend, [
+     		    {max_open_files, 50},
+       			{data_root, "/var/lib/riak/leveldb"}
+  	        ]},
+     	       	  {be_blocks, riak_kv_bitcask_backend, [
+  		     {data_root, "/var/lib/riak/bitcask"}
+ 	        ]}
+ 	    ]},
             %% raw_name is the first part of all URLS used by the Riak raw HTTP
             %% interface.  See riak_web.erl and raw_http_resource.erl for
             %% details.
```
