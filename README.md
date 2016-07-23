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
    - node1
      - riak/riak-cs/stanchion
    - node2
      - riak/riak-cs
    - node3
      - riak/riak-cs 


- riak/riak-cs/stanchionは以下の場所からダウンロードする
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
# wget http://s3.amazonaws.com/downloads.basho.com/riak/1.4/1.4.1/rhel/6/riak-1.4.1-1.el6.x86_64.rpm
# wget http://s3.amazonaws.com/downloads.basho.com/stanchion/1.4/1.4.0/rhel/6/stanchion-1.4.0-1.el6.x86_64.rpm
# wget http://s3.amazonaws.com/downloads.basho.com/riak-cs/1.4/1.4.0/rhel/6/riak-cs-1.4.0-1.el6.x86_64.rpm

# rpm -ivh riak-1.4.1-1.el6.x86_64.rpm
# rpm -ivh riak-cs-1.4.0-1.el6.x86_64.rpm
# rpm -ivh stanchion-1.4.0-1.el6.x86_64.rpm

# cp -pr app.config app.config.org
```


### Riakを設定変更する
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

- `/etc/riak/vm.args`を編集する
```
# diff -u vm.args.org vm.args
--- vm.args.org	2013-08-01 21:29:20.000000000 +0000
+++ vm.args	2016-07-23 12:43:34.214035178 +0000
@@ -1,5 +1,5 @@
 ## Name of the riak node
--name riak@127.0.0.1
+-name riak@192.168.33.10

 ## Cookie for distributed erlang.  All nodes in the same cluster
 ## should use the same cookie or they will not be able to communicate.
```

### Riak-CSの設定変更
- `/etc/riak-cs/app.config`を設定する
```
# diff -u app.config.org app.config
--- app.config.org	2013-08-12 18:33:52.000000000 +0000
+++ app.config	2016-07-23 12:52:29.092037758 +0000
@@ -7,16 +7,16 @@

               %% Riak CS http/https port and IP address to listen at
               %% for object storage activity
-              {cs_ip, "127.0.0.1"},
+              {cs_ip, "192.168.33.10"},
               {cs_port, 8080 } ,

               %% Riak node to which Riak CS accesses
-              {riak_ip, "127.0.0.1"},
+              {riak_ip, "192.168.33.10"},
               {riak_pb_port, 8087 } ,

               %% Configuration for access to request
               %% serialization service
-              {stanchion_ip, "127.0.0.1"},
+              {stanchion_ip, "192.168.33.10"},
               {stanchion_port, 8085 },
               {stanchion_ssl, false },

@@ -52,7 +52,7 @@
               %% Root host name which Riak CS accepts.
               %% Your CS bucket at s3.example.com will be accessible
               %% via URL like http://bucket.s3.example.com/object/name
-              {cs_root_host, "s3.amazonaws.com"},
+              {cs_root_host, "riak-dev.hoge.jp"},

               %% Connection pools
               %% Each pool is specified as a nested
```
- `/etc/riak-cs/vm.args`を設定する
```
# diff -u vm.args.org vm.args
--- vm.args.org	2013-08-12 18:33:52.000000000 +0000
+++ vm.args	2016-07-23 12:54:37.060040749 +0000
@@ -1,5 +1,5 @@
 ## Name of the riak node
--name riak-cs@127.0.0.1
+-name riak-cs@192.168.33.10

 ## Cookie for distributed erlang.  All nodes in the same cluster
 ## should use the same cookie or they will not be able to communicate.
```

### Stanchionを設定変更する
- `/etc/stanchion/app.config`を設定する
```
# diff -u app.config.org app.config
--- app.config.org	2013-08-12 19:52:40.000000000 +0000
+++ app.config	2016-07-23 13:00:38.677033725 +0000
@@ -3,7 +3,7 @@
 [
  %% Stanchion config
  {stanchion, [
-                   {stanchion_ip, "127.0.0.1"},
+                   {stanchion_ip, "192.168.33.10"},
                    {stanchion_port, 8085 } ,

                    %%{ssl, [
@@ -14,7 +14,7 @@
                    {auth_bypass, false } ,

                    %% Riak connection details
-                   {riak_ip, "127.0.0.1"},
+                   {riak_ip, "192.168.33.10"},
                    {riak_pb_port, 8087 },

                    %% Admin user credentials
```
- `/etc/stanchion/vm.args`を設定する
```
# diff -u vm.args.org vm.args
--- vm.args.org	2013-08-12 19:52:40.000000000 +0000
+++ vm.args	2016-07-23 13:03:57.399033400 +0000
@@ -1,5 +1,5 @@
 ## Name of the riak node
--name stanchion@127.0.0.1
+-name stanchion@192.168.33.10

 ## Cookie for distributed erlang.  All nodes in the same cluster
 ## should use the same cookie or they will not be able to communicate.
```

### Riak/Riak-CS/Stanchionの起動
- 各nodeで実行(stanchionの起動はnode1だけ)
```
# riak start
# riak ping
pong
# stanchion start
# stanchion ping
pong
```

- node2,node3からリングへ参加する
```
# riak-admin cluster join riak@192.168.33.10

# riak-admin cluster plan
# riak-admin cluster commit
```

- node2,node3が参加していない状態(node1から確認)
```
# riak-admin member_status
================================= Membership ==================================
Status     Ring    Pending    Node
-------------------------------------------------------------------------------
valid     100.0%      --      'riak@192.168.33.10'
-------------------------------------------------------------------------------
Valid:1 / Leaving:0 / Exiting:0 / Joining:0 / Down:0
```


- node2,3がリングへ参加した状態(node1から確認)
```
# riak-admin member_status
================================= Membership ==================================
Status     Ring    Pending    Node
-------------------------------------------------------------------------------
valid      34.4%      --      'riak@192.168.33.10'
valid      32.8%      --      'riak@192.168.33.11'
valid      32.8%      --      'riak@192.168.33.12'
-------------------------------------------------------------------------------
Valid:3 / Leaving:0 / Exiting:0 / Joining:0 / Down:0
```

### 管理者用のAccess_Keyを作成する
