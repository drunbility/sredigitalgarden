

### 基础概述
#todo 

RPC(Remote Procedure Call)在HDFS中应用广泛，无论是来自Client的请求，或是集群间资源调度。它以Protocol Buffers为基础，

hadoop 直接使用 protocol buffers 利用动态代理,实现了一套类似 grpc 的 rpc 框架


### 重要参数

`ipc.maximum.data.length`


---
[HadoopRpc - HADOOP2 - Apache Software Foundation](https://cwiki.apache.org/confluence/display/HADOOP2/HadoopRpc)
