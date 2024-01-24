安装使用
==================


编译
--------

环境依赖

1. CentOS、ubuntu和Mac OS都支持（推荐CentOS >= 7.2）
2. go >= 1.19
3. gcc >= 5（使用scann模型时，gcc >= 9）
4. cmake >= 3.17 
5. OpenBLAS
6. tbb，CentOS可使用yum安装，如：yum install tbb-devel.x86_64
7. [RocksDB](https://github.com/facebook/rocksdb) == 6.2.2 （可选），你不需要手动安装，脚本自动安装。但是你需要手动安装rocksdb的依赖。请参考如下安装方法：https://github.com/facebook/rocksdb/blob/master/INSTALL.md
8. CUDA >= 9.0，如果你不使用GPU模型，可忽略。
9. clang >= 8.0，bazel, python >= 3.7,如果你不使用scann模型，可忽略。

编译

-  进入 `GOPATH` 目录, `cd $GOPATH/src` `mkdir -p github.com/vearch` `cd github.com/vearch`

-  下载源代码: ``git clone https://github.com/vearch/vearch.git`` (后续使用$vearch
   代表vearch目录绝对路径)

-  如果使用GPU版本, 修改$vearch/engine/CMakeLists.txt文件中BUILD_WITH_GPU变为on

-  如果使用SCANN模型, 修改$vearch/engine/CMakeLists.txt文件中BUILD_WITH_SCANN变为on

-  编译vearch和gamma

   1. ``cd build``
   2. ``sh build.sh``
   
   生成\ ``vearch``\ 文件表示编译成功

部署
--------

单机模式:

-  生成配置文件config.toml(master_server端口使用8817, router_server端口使用9001）
::

   [global]
       # the name will validate join cluster by same name
       name = "vearch"
       # you data save to disk path ,If you are in a production environment, You'd better set absolute paths
       data = ["datas/"]
       # log path , If you are in a production environment, You'd better set absolute paths
       log = "logs/"
       # default log type for any model
       level = "info"
       # master <-> ps <-> router will use this key to send or receive data
       signkey = "vearch"
       skip_auth = true

   # if you are master, you'd better set all config for router、ps and router, ps use default config it so cool
   [[masters]]
       # name machine name for cluster
       name = "m1"
       # ip or domain
       address = "127.0.0.1"
       # api port for http server
       api_port = 8817
       # port for etcd server
       etcd_port = 2378
       # listen_peer_urls List of comma separated URLs to listen on for peer traffic.
       # advertise_peer_urls List of this member's peer URLs to advertise to the rest of the cluster. The URLs needed to be a comma-separated list.
       etcd_peer_port = 2390
       # List of this member's client URLs to advertise to the public.
       # The URLs needed to be a comma-separated list.
       # advertise_client_urls AND listen_client_urls
       etcd_client_port = 2370
       
   [router]
       # port for server
       port = 9001
   
   [ps]
       # port for server
       rpc_port = 8081
       # raft config begin
       raft_heartbeat_port = 8898
       raft_replicate_port = 8899
       heartbeat-interval = 200 #ms
       raft_retain_logs = 10000
       raft_replica_concurrency = 1
       raft_snap_concurrency = 1 

-  启动

启动vearch前，需要设置LD_LIBRARY_PATH环境变量的值，添加gamma, rocksdb, zfp等依赖的lib包; 执行ldd vearch 和 ldd $vearch/ps/engine/gammacb/lib/lib/libgamma.so.0.1 命令可以查看vearch和gamma依赖的包)。

::

   ./vearch -conf conf.toml all



集群模式:  

- vearch 有三个模块: ``ps``, ``master``, ``router``, run ``./vearch -conf config.toml ps/router/master`` 启动相应模块

假如有5台机器， 2台作为master管理， 2台作为ps计算节点， 1台router请求转发

-  master

   -  192.168.1.1
   -  192.168.1.2

-  ps

   -  192.168.1.3
   -  192.168.1.4

-  router

   -  192.168.1.5


-  生成toml格式配置文件 config.toml， 作为master的机器ip配置在[[masters]]中，支持多个，router和ps所在机器ip无需配置。

::

    [global]
        name = "vearch"
        data = ["datas/"]
        log = "logs/"
        level = "info"
        signkey = "vearch"
        skip_auth = true

    # if you are master, you'd better set all config for router、ps and router, ps use default config it so cool
    [[masters]]
        name = "m1"
        address = "192.168.1.1"
        api_port = 8817
        etcd_port = 2378
        etcd_peer_port = 2390
        etcd_client_port = 2370
    [[masters]]
        name = "m2"
        address = "192.168.1.2"
        api_port = 8817
        etcd_port = 2378
        etcd_peer_port = 2390
        etcd_client_port = 2370
    [router]
        port = 9001
        skip_auth = true
    [ps]
        rpc_port = 8081
        raft_heartbeat_port = 8898
        raft_replicate_port = 8899
        heartbeat-interval = 200 #ms
        raft_retain_logs = 10000
        raft_replica_concurrency = 1
        raft_snap_concurrency = 1
        
-  启动vearch前，设置LD_LIBRARY_PATH环境变量加载依赖包

-  on 192.168.1.1 , 192.168.1.2 run master

::

    ./vearch -conf config.toml master

-  on 192.168.1.3 , 192.168.1.4 run ps

::

    ./vearch -conf config.toml ps

-  on 192.168.1.5 run router

::

    ./vearch -conf config.toml router

