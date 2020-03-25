## 基础知识
### 分布式模式启动zookeeper脚本
-- zks-daemons.sh start
```
#! /bin/bash
#设置Zookeeper集群节点地址
hosts=( dn1 dn2 dn3 )

# 获取输入zookeeper命令参数
cmd=$1
#执行分布式管理命令
function zookeeper(){
  for i in ${hosts[@]}
    do 
       ssh  testuser@$i "source ~/.bash_profire; zkServer.sh $cmd; echo zookeeper node is $i,run $cmd command." & sleep 1
    done
}

#判断输入的zoopkeeper命令参数是否有效
case "$1" in
  start)
     zookeeper
     ;;
   stop)
     zookeeper
     ;;
   status)
     zookeeper
     ;;
   start-foreground)
     zookeeper
     ;;
   upgrade)
     zookeeper
     ;;
   restart)
     zookeeper
     ;;
   pring-cmd)
     zookeeper
     ;;
   *)
     echo "Usage:$0{start|stop|status|start-foreground|upgrade|restart|pring-cmd}"
     RETVAL=1
   esac
```

### 分布式模式启动kafka脚本
```
#! /bin/bash
#Kafka集群节点地址
hosts=( dn1 dn2 dn3 )

mill=`date "+%N"`
tdate=`date "+%Y-%m-%d %H:%M:%S, ${mill:0:3}"`

echo [$tdate] INFO [Kafka Cluster] begins to execute the $1 operation.

#执行分布式开启命令
function start(){
  for i in ${hosts[@]}
     do 
        smill=`date "+%N"`
        stdate=`date "+%Y-%m-%d %H:%M:%S, ${smill:0:3}"`
        ssh test@i "source ~/.bash_profire; echo [$stdate] INFO [Kafka Broker $i]
        begins to execute the startup operation.;kafka-server-start.sh $KAFKA_HOME/config/server.properties >/dev/null;" & sleep 1
     done
}

function stop(){
  for i in ${hosts[@]}
     do 
        smill=`date "+%N"`
        stdate=`date "+%Y-%m-%d %H:%M:%S, ${smill:0:3}"`
        ssh test@i "source ~/.bash_profire; echo [$stdate] INFO [Kafka Broker $i]
        begins to execute the shutdown operation.;kafka-server-stop.sh >/dev/null;" & sleep 1
     done
}


function status(){
  for i in ${hosts[@]}
     do 
        smill=`date "+%N"`
        stdate=`date "+%Y-%m-%d %H:%M:%S, ${smill:0:3}"`
        ssh test@i "source ~/.bash_profire; echo [$stdate] INFO [Kafka Broker $i]
        status message is :;jps |grep Kafka; " & sleep 1
     done
}


case "$1" in 
  start)
    start
    ;;
  stop)
    stop
    ;;
  status)
    status
    ;;
  *)
    echo "Usage:$0 {start|stop|status}"
    RETVAL=1
esca
```
### 创建主题
  - 自动创建
    - 通过auto.create.topics.enable=true来自动创建主机
  - 手动创建
  ```
  kafka-topics.sh --create -zookeeper ip1:2181,ip2:2181,ip3:2181 --replication-factor 3 --partitions 6 --topic test
  
  #访问zookeeper
  zkCli.sh -server ip1:2181,ip2:2181,ip3:2181
  #执行ls命令查看主题分区数
  ls /brokers/topics/test/partitions
  
  #使用get命令查看分区元数据信息
  get /brokers/topics/test
  ```
  
### 查看主题
  - describe命令:用来查看指定主题或全部主题的详细信息
  - list命令:用来展示所有的主题名
  ```
  #查看单个
  kafka-topics.sh --describe -zookeeper ip1:2181,ip2:2181,ip3:2181 --topic test
  
  #查看全部
  kafka-topics.sh --describe -zookeeper ip1:2181,ip2:2181,ip3:2181
  
  #查看所有主题名
  kafka-topics.sh --list -zookeeper ip1:2181,ip2:2181,ip3:2181
  ```
  - 查看正在同步的主题
  ```
  #查看全部
  kafka-topics.sh --describe -zookeeper ip1:2181,ip2:2181,ip3:2181 --under-replicated-partitions
  #查看特定
  kafka-topics.sh --describe -zookeeper ip1:2181,ip2:2181,ip3:2181 --topic test --under-replicated-partitions
  ```
  - 查看主题中不可用的分区
  ```
  #查看全部
  kafka-topics.sh --describe -zookeeper ip1:2181,ip2:2181,ip3:2181 --unavailable-partitions
  #查看特定
  kafka-topics.sh --describe -zookeeper ip1:2181,ip2:2181,ip3:2181 --topic test --unavailable-partitions
   ```
  - 查看主题重写的配置
  ```
  #查看全部
  kafka-topics.sh --describe -zookeeper ip1:2181,ip2:2181,ip3:2181 --topic-with-overrides
  #查看特定
  kafka-topics.sh --describe -zookeeper ip1:2181,ip2:2181,ip3:2181 --topic test --topic-with-overrides
  ```

### 修改主题
  - 主题可以通过alter命令进行修改，修改内容包含主题的配置信息
  ```
  #创建主题
  kafka-topics.sh --create -zookeeper ip1:2181,ip2:2181,ip3:2181 --replication-factor 1 --partitions 1 --topic user_order2 
  --config max.message.bytes=102400
  #查看可覆盖参数
  kafka-topics.sh --describe -zookeeper ip1:2181,ip2:2181,ip3:2181 --topic user_order --topic-with-overrieds
  #使用alter和config修改配置参数
  kafka-topics.sh --alter -zookeeper ip1:2181,ip2:2181,ip3:2181 --topic user_order --config max.message.bytes=102400
  ```
  
### 删除主题
