安装使用
==================


编译
--------

环境依赖

1. 支持CentOS, Ubuntu and Mac OS。推荐CentOS 7.2以上， 检查安装cmake、make工具。
2. Go版本>=1.11.2。
3. Gcc版本>=5。
4. `Faiss <https://github.com/facebookresearch/faiss>`_ 版本>=v1.6.0(不建议使用最新的faiss版本)。
5. 如果使用RocksDB， 版本6.2.2, 使用RocksDB中INSTALL.md文件的make shared_lib进行编译。
6. 如果使用GPU, CUDA版本>=9.0

编译

-  进入GOPATH工作目录， ``cd $GOPATH/src`` ``mkdir -p github.com/vearch`` ``cd github.com/vearch``

-  下载源代码: ``git clone https://github.com/vearch/vearch.git`` (后续使用$vearch
   代表vearch目录绝对路径)

-  下载gamma代码: ``cd $vearch/engine`` ``git clone https://github.com/vearch/gamma.git``

-  如果使用GPU版本, 修改$vearch/engine/gamma/CMakeLists.txt文件中BUILD_WITH_GPU值为on

-  编译gamma

   1. ``cd $vearch/engine/gamma``
   2. ``mkdir build && cd build``
   3. ``export FAISS_HOME=faiss安装路径``
   4. 如果使用RocksDB ``export ROCKSDB_HOME=RocksDB安装路径``
   5. ``cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$vearch/ps/engine/gammacb/lib ..``
   6. ``make && make install``

-  编译vearch

   1. ``cd $vearch``
   2. ``export FAISS_HOME=faiss安装路径``
   3. 如果使用RocksDB ``export ROCKSDB_HOME=RocksDB安装路径``
   4. ``export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$FAISS_HOME/lib:$vearch/ps/engine/gammacb/lib/lib`` 
   5. ``go build -a --tags=vector -o  vearch``
   
   生成\ ``vearch``\ 文件表示编译成功

部署
--------

单机模式:

-  生成配置文件config.toml (toml格式, master_server端口使用8817, router_server端口使用9001）
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

启动vearch前，需要设置LD_LIBRARY_PATH环境变量的值，添加faiss, gamma, rocksdb等依赖的lib包(比如$vearch/ps/engine/gammacb/lib/lib、$FAISS_HOME/lib等目录下的lib包; 执行ldd vearch 和 ldd $vearch/ps/engine/gammacb/lib/lib/libgamma.so.0.1 命令可以查看vearch和gamma依赖的包)。

::

   ./vearch -conf config.toml



集群模式:  

- vearch 有三个模块: ``ps``, ``master``, ``router``, run ``./vearch -f config.toml ps/router/master`` 启动相应模块

假如有5台机器， 2台作为master管理， 2台作为ps计算节点， 1台请求转发

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

