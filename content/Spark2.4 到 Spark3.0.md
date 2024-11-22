
### Spark2.4 到 Spark3.0

#### Spark 兼容相关

- 兼容旧的 external shuffle service , `set spark.shuffle.useOldFetchProtocol=true` 否则会报错`IllegalArgumentException: Unexpected message type: <number>`
    
- 如果升级了 history server ,读旧的任务 event log ,可能会有乱码问题. 需要在旧版本中指定 spark driver 和 executor 的编码. 这样之后 spark 3 history server 就不会乱码
    
    - spark.driver.extraJavaOptions="-Dfile.encoding=utf-8"
        
    - spark.executor.extraJavaOptions="-Dfile.encoding=utf-8"
        
- `spark.sql.legacy.createHiveTableByDefault` 设置为 false ,默认为 true . 设置为 false 后,表的默认 provider 为 parquet ,默认存储格式由 `spark.sql.sources.default` 控制. 为 true 时,表的默认 provider 为 hive ,默认存储格式退化到 hive 的配置文件来控制,如需在 spark的配置文件中配置,需要加上 spark.hadoop 前缀`spark.hadoop.hive.default.fileformat`
    
    - 已知问题当 provider 为 hive 时,在进行覆盖写入的时候,有些 spark 自身控制逻辑会失效.比如参数`spark.sql.sources.partitionOverwriteMode`
        
    - 为了避免未知问题,还是保持默认行为,不修改配置 `spark.sql.legacy.createHiveTableByDefault` ,指定`spark.hadoop.hive.default.fileformat` 为 parquet
        
- ==spark 3 中 ,非 hive-serde 表中不允许 char 类型的字段, 所以相关的 create ,alter 语句会失败,需要强制改用 string 类型.这个行为是跟 Spark2 中不同的,在Spark 2.4及以下版本中，CHAR类型被视为STRING类型，并且长度参数被简单地忽略。(hudi 表可能会面临这个校验)==
    
- 在 Spark 2.4 中,分区列值和表本身的 schema 定义冲突的时候,这个分区列值默认会转换为 null. 到了 Spark 3 版本中,首先会进行强校验,如果校验不通过, SQL 语句会抛出异常. 可以通过设置 `spark.sql.sources.validatePartitionColumns` 关掉校验.但还是建议不设置,来暴露这些有问题的分区列值
    
- 关于参数 `spark.sql.hive.convertMetastoreOrc` 和 `spark.sql.hive.convertMetastoreParquet` 自 Spark 3.0 后, Spark 实现了自己的 parquet 和 orc 文件的 reader 和 writer .在之前是复用 hive 自身的 reader 和 writer . 这里就会造成某些特定场景下(比如非分区表 insert overwrite 覆盖自身)的报错. 这两个值默认为 true,可以把两个值都设置成 false ,能够规避某些数据任务SQL语句报错的情况. 但按照官方说明,为 true会提升性能,并且一般情况下为 true 不会造成执行报错. 建议在数据任务模块里面设置这两个参数为 false ,不要放在 spark-default.conf 文件内.==后续 hudi 表可能会需要 spark 自身的 parquet reader writer==
    
    * Relation conversion from metastore relations to data source relations for better performance  
     *  
     * - When writing to non-partitioned Hive-serde Parquet/Orc tables  
     * - When scanning Hive-serde Parquet/ORC tables  
       
    ​
     ^9a694d
- 在 Spark 3 中,truncate table 操作会去对路径做 setPermission 操作,在 hdfs 中,不是 owner 是不能进行这个操作的, Spark 程序会抛出权限拒绝异常.这个是 Spark3 中的逻辑,可以设置 `spark.sql.truncateTable.ignorePermissionAcl.enabled=true` 跳过这个逻辑
    

#### 资源相关配置

1. 动态资源申请 和之前保持一致
    
2. AQE 动态选择 Join 策略
    
    spark.sql.adaptive.autoBroadcastJoinThreshold = 128m
    

3. AQE 自动分区合并
    
    spark.sql.adaptive.coalescePartitions.enabled=true  
    spark.sql.adaptive.advisoryPartitionSizeInBytes=128m  
    spark.sql.adaptive.coalescePartitions.minPartitionSize=64m  
    spark.sql.adaptive.coalescePartitions.initialPartitionNum=512  
    spark.sql.adaptive.coalescePartitions.parallelismFirst=false
    
4. AQE 倾斜自动处理
    
    spark.sql.adaptive.skewJoin.enabled=true  
    spark.sql.adaptive.skewJoin.skewedPartitionFactor=5  
    spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes=1024m
    

#### 框架级别依赖和任务级别依赖管理

spark.yarn.archive 和 spark.yarn.jars 是配置 spark 自身依赖的 .如果配置spark.yarn.archive ,会替换 spark.yarn.jars,网上有测试 spark.yarn.jars 性能比 spark.yarn.archive 好点

spark.yarn.jars hdfs:``///tmp/spark-yarn/jar/*.jar

spark-submit --jars 配置任务级别的依赖,是通过 SparkContext.addjar 方法合并到框架级别依赖里面去的

#### 配置修改管理

Spark SQL 配置有两种:一种是运行时配置,一种是静态配置. 运行时配置是可以在 session 级别动态修改的,并且同样的 key session 级别的修改是会覆盖前面的设置

静态配置是在 spark-submit 通过 --conf 传入的,不能在 session 级别动态修改

要注意在 spark3 中在 session 中通过 set 语句来修改静态配置,会抛出一个异常,如果提交的程序没有处理的话,程序会中断

org.apache.spark.sql.AnalysisException: Cannot modify the value of a Spark config:

这种 exception 无法跟其他 sql 语句解析异常区分,当表不存在时也会报 AnalysisException

可以通过关掉检查来避免抛出异常 :

spark.sql.legacy.setCommandRejectsSparkCoreConfs = false

#### 边缘 case

- spark.sql.storeAssignmentPolicy ANSI, legacy and strict 设置强转策略,Spark2.x 中是最宽松的策略 ,即 legacy , Spark3.x 默认为 ANSI 策略,次宽松策略.strict 策略禁止强转
#### SQL 函数相关

^6390c0

any_value 函数不支持 

1. 现在用的 spark  3.2.4 版本还没有实现这个函数
2. 可以把 any_value 替换成其他函数，比如 first_value 或者 last_value

