#泄漏

### 现象
用户反馈 hs2 打开的文件描述符的数量一直在涨，但是当前 hs2 的连接只有个位数。

![[wecom-temp-86668-466aa580e0ca33aef0dd18c39de36447.png]]


### 排查过程

首先找到 hs2 进程持有了哪些文件描述符,通过 [[shell 常用查找删除命令#^6396f2|lsof 命令]]  `lsof -p $pid`   ,看到 hs2 进程确实在 `/data/emr/hive/tmp/operation_logs/` 目录下打开了大量描述符

在  jira 中找到一个类似 的 issue: [[HIVE-10970] Investigate HIVE-10453: HS2 leaking open file descriptors when using UDFs - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HIVE-10970)

但是这个场景是由于 UDF 导致的 fd 泄漏,并且泄漏路径是在  `hive.downloaded.resources.dir` 路径下,跟 operation_logs 目录不一样.看上去不是同一个问题


排查源码 , 找到 operation log 有一个清理逻辑
`org.apache.hive.service.cli.operation.Operation#cleanupOperationLog`

猜测是在客户端 session [[hive 中的临时目录#^a01142|异常结束]] 的时候,这个方法没有==被正常调用到或者清理逻辑有漏洞==导致的

首先过一遍 session 关闭的逻辑,通过分析 beeline 客户端的火焰图,找到 session 关闭起始点
`org.apache.hive.jdbc.HiveStatement#closeClientOperation`
![[Pasted image 20230303195911.png]]

这里 client 发起了一个 thrift rpc 调用,然后在 hs2 thrift 找到 thrift server 对应的方法 `org.apache.hive.service.cli.thrift.ThriftCLIService#CloseOperation`
跟踪这个方法,最终会走到 `org.apache.hive.service.cli.operation.SQLOperation#close`
这里会调用 cleanupOperationLog 方法
![[Pasted image 20230303200607.png]]


那么确实是有可能由于客户端 session 异常退出,operation logs 没有被清理的可能的


接着查看 cleanupOperationLog 逻辑, 看这里是否有代码 bug ,于是在 idea 中使用 git 分支比较功能,发现 3.1 版本提交了一个修复


![[Pasted image 20230303193129.png]]


[[HIVE-18820] Operation doesn't always clean up log4j for operation log - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HIVE-18820)





### 结论

- 客户端 session 异常退出,导致 operation logs 没有被清理,跟 scratch dir 没有被清理场景类似
- HIVE-18820 社区 bug 导致,可以考虑合入这个 patch 



