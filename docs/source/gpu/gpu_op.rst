GPU使用
=================


注意事项
-----------------------

1. 编译环境需要带GPU安装GUDA>=9.0.

2. 编译前Vearch引擎CMakeLists.txt配置中BUILD_WITH_GPU设置为on.

3. 创建表时gamm设置参数nprobe不大于1024(CUDA9.0)或者不大于2048(CUDA>=9.2), index_size设置为0.

4. 建表时特征字段retrieval_type参数设置为GPU.

5. 数据插入和建索引时不支持搜索。

6. 数据不自动创建索引，调用curl -XPOST http://router_server/$db_name/$space_name/_forcemerge创建索引。

7. 不支持实时添加数据到GPU索引，新增数据只有更新索引后才会生效。(更新索引:curl -XPOST http://router_server/$db_name/$space_name/_forcemerge)。

