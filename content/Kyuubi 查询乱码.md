

数据是经过如下路径被读到即席查询的:

hdfs file ->  yarn spark  executor ->  yarn spark driver -> kyuubi server spark submit client ->  kyuubi server client( dbeaver , beeline cli, connector druid.... )

根据这个流程来逐步排除:

1. 使用 hive shell  去查表,没有乱码,说明数据本身是 utf8 编码,排除了数据本身问题
2. 使用 beeline 去直连 kyuubi server 去查,kyuubi 新建 session ,查询没有乱码,说明这种场景下, kyuubi server 能正常解析数据 
3. 使用 beeline 用 zk url 方式去连接 kyuubi server 去查,新建 session ,查询没有乱码,说明这种场景下, kyuubi server 能正常解析数据 
4. 上面基本排除 kyuubi server 服务启动的时候,没有引入错误的编码格式
5. 使用 dbeaver 用 zk url 方式去连接 kyuubi server 去查,新建 session,查询乱码
6. 代码使用 jdbc 代码 用 zk url 方式去连接 kyuubi server 去查,新建 session ,查询乱码
7. beeline 使用 zk url 方式去连接 kyuubi server ,指定第五步生成的 session ,查询乱码
8. 上面基本确认是新建 session 时,某种差异查询导致的问题
9. 接着排除两种 session 的差异,比较两个 session 的配置,发现一个关键配置差异 `spark.sql.hive.convertMetastoreOrc ` ,验证该参数为 true 时正常,为 false 时查询乱码
10. `spark.sql.hive.convertMetastoreOrc` 这个参数默认为 true ,当时为了解决即席查询数据越界错误而改成 false .

解决方案:

之前数据越界问题是数据本身问题,需要重刷数据来解决,修改参数只是零时解决方案,并且会造成乱码问题, 所以 spark.sql.hive.convertMetastoreOrc  [[Spark2.4 到 Spark3.0#^9a694d]]] 参数修改要回滚来避免乱码问题

