集群实验
--------

集群实验分别对 VGG100M , VGG500M 和 VGG1B进行实验，并添加是否过滤来进行实验，其中过滤是指在搜索的时候指定过滤条件来缩小搜索范围。VGG100M 搭建了 3 个masters, 3个 routers 和5 个 partition services 的集群。 VGG500M搭建了 3 个masters, 3个 routers 和24个 partition services 的集群。VGG1B搭建了 3 个masters, 6个 routers 和48 个 partition services 的集群。

**结果**

.. image:: ../pic/cluster.png
   :align: center
   :scale: 100 %
   :alt: Architecture


可以看到当average latency超过一定程度，QPS就不再发生明显变化了。

