



spark 即席查询要开启分区表严格限制
- spark.hadoop.hive.mapred.mode=strict   严格模式总开关,Hive Strict Mode 用来阻止“高风险 / 易拖垮集群”的 SQL 写法，典型目标是：全表扫描、全局排序、笛卡尔积、隐式类型错误等。
- spark.hadoop.hive.strict.checks.no.partition.filter = true  强制查询分区表时必须指定分区条件
- spark.hadoop.hive.strict.checks.orderby.no.limit = true  禁止 ORDER BY 无 LIMIT
- spark.hadoop.hive.strict.checks.cartesian.product=true 禁止笛卡尔积