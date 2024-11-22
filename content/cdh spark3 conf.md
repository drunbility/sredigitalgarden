  

spark.yarn.jars hdfs:///tmp/spark-yarn/jar/*.jar

  
### legacy

```
spark.shuffle.useOldFetchProtocol  true

#spark.sql.legacy.createHiveTableByDefault false

spark.hadoop.hive.default.fileformat=parquet

spark.sql.legacy.setCommandRejectsSparkCoreConfs false

spark.sql.truncateTable.ignorePermissionAcl.enabled true

spark.hadoop.hive.exec.dynamic.partition=true

spark.hadoop.hive.exec.dynamic.partition.mode=nonstrict

  

#spark.sql.hive.convertMetastoreOrc false

#spark.sql.hive.convertMetastoreParquet  false
```
  

# DRA for test

```
spark.dynamicAllocation.enabled=true

spark.dynamicAllocation.executorIdleTimeout=60

spark.dynamicAllocation.maxExecutors=10

spark.dynamicAllocation.minExecutors=0

spark.dynamicAllocation.schedulerBacklogTimeout=3

spark.dynamicAllocation.sustainedSchedulerBacklogTimeout=3

  ```
  
  

## DRA for pro

```

#spark.dynamicAllocation.enabled=true

#spark.dynamicAllocation.executorIdleTimeout=60

#spark.dynamicAllocation.maxExecutors=30

#spark.dynamicAllocation.minExecutors=2

#spark.dynamicAllocation.schedulerBacklogTimeout=3

#spark.dynamicAllocation.sustainedSchedulerBacklogTimeout=3

  

spark.eventLog.enabled=true

spark.eventLog.compress=true

spark.eventLog.dir=hdfs://devns01/user/spark/applicationHistory

spark.eventLog.rolling.enabled true

spark.eventLog.rolling.maxFileSize=512m

spark.eventLog.buffer.kb=10m

spark.driver.log.dfsDir /user/spark/driverLogs

spark.driver.log.persistToDfs.enabled=true

  

spark.yarn.historyServer.address=master02:18089

spark.history.ui.port=18089

spark.yarn.historyServer.allowTracking=true

  
  

spark.serializer=org.apache.spark.serializer.KryoSerializer

spark.shuffle.service.enabled=true

spark.shuffle.service.port=7337

  
  

spark.driver.extraJavaOptions=-Dfile.encoding=utf-8

spark.executor.extraJavaOptions=-Dfile.encoding=utf-8

spark.master=yarn

spark.submit.deployMode=client

  

spark.sql.hive.metastore.version=2.1.1

spark.sql.hive.metastore.jars=/opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/lib/hive/lib/*

  

spark.driver.extraLibraryPath=/opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/lib/hadoop/lib/native

spark.executor.extraLibraryPath=/opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/lib/hadoop/lib/native

spark.yarn.am.extraLibraryPath=/opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/lib/hadoop/lib/native

spark.yarn.config.gatewayPath=/opt/cloudera/parcels

spark.yarn.config.replacementPath={{HADOOP_COMMON_HOME}}/../../..

  

spark.yarn.appMasterEnv.MKL_NUM_THREADS=1

spark.executorEnv.MKL_NUM_THREADS=1

spark.yarn.appMasterEnv.OPENBLAS_NUM_THREADS=1

spark.executorEnv.OPENBLAS_NUM_THREADS=1
```
  
  

# not work delete

```
#spark.lineage.log.dir=/var/log/spark/lineage

#spark.lineage.enabled=true

#spark.extraListeners=com.cloudera.spark.lineage.NavigatorAppListener

#spark.sql.queryExecutionListeners=com.cloudera.spark.lineage.NavigatorQueryListener
```
  
  

### new AQE

^0ce293

  
```

spark.sql.adaptive.autoBroadcastJoinThreshold = 64m

  

spark.sql.adaptive.coalescePartitions.enabled=true

spark.sql.adaptive.advisoryPartitionSizeInBytes=256m

spark.sql.adaptive.coalescePartitions.minPartitionSize=128m

#spark.sql.adaptive.coalescePartitions.initialPartitionNum=512

spark.sql.adaptive.coalescePartitions.parallelismFirst=false

  

spark.sql.adaptive.skewJoin.enabled=true

spark.sql.adaptive.skewJoin.skewedPartitionFactor=5

spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes=512m
```
