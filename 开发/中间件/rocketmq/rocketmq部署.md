<center><h1>rocketmq部署</h1></center>

# 1.双机房双节点

节点A 周浦（10.249.136.93）

节点B 南汇（10.249.130.84）

## 1.1.修改安装和数据保存磁盘

使用lsblk命令查看/opt目录所使用的盘符名,如下图

![img](https://img-blog.csdnimg.cn/2019050609392583.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NDM0NTcx,size_16,color_FFFFFF,t_70)

将盘符名替换/opt/rocketmq-all-4.3.2-bin-release/os.sh脚本中的$DISK后使用root执行该脚本；

# 2.安装rocketmq

### 2.1.从mirrors.deppon.com介质库的mq/RocketMQ4.3目录中下载jdk8.0_201.zip与rocketmq-all-4.3.2-bin-release.tar.gz，上传到所需安装的每个分区的的/opt目录下

![img](https://img-blog.csdnimg.cn/20190506094123362.png)

### 2.2.解压介质,执行

unzip jdk1.8.0_201.zip

tar zxvf rocketmq-all-4.3.2-bin-release.tar.gz

### 2.3.修改java介质执行权限，执行

chmod -R 775 jdk1.8.0_201

### 2.4.配置java环境变量

(appman用户)，执行vi /home/appman/.bash_profile

加入以下内容并保存

export JAVA_HOME=/opt/jdk1.8.0_201

export PATH=$JAVA_HOME/bin:$PATH

export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

![img](https://img-blog.csdnimg.cn/20190506094318369.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NDM0NTcx,size_16,color_FFFFFF,t_70)6.

### 2.5.验证java环境

使用appman用户重新登录，执行java –version，显示如图则成功

![img](https://img-blog.csdnimg.cn/20190506094352870.png)

# 3.修改配置文件

### 3.1.修改broker启动脚本的内存设置

修改/opt/rocketmq-all-4.3.2-bin-release/bin/runbroker.sh启动文件中的最大最小内存为4G，年轻代内存为2G，如图：

![img](https://img-blog.csdnimg.cn/20190506094432465.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NDM0NTcx,size_16,color_FFFFFF,t_70)

### 3.2.RocketMQ配置

修改properties配置文件

cd /opt/rocketmq-all-4.3.2-bin-release/conf/2m-2s-async

broker-a.properties:主节点配置文件

broker-a-s.properties:从节点配置文件

![img](https://img-blog.csdnimg.cn/20190506094542813.png)

-------------------------------------------------主要是这两个配置文件--------------------------------------------------------

1. 主节点配置修改

   ![img](https://img-blog.csdnimg.cn/20190506094619461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NDM0NTcx,size_16,color_FFFFFF,t_70)

2. BrokerName规范：

   ​              第一个主节点名为c1-m1，第二个主节点名为c1-m2，以此类推。

   ​              如一个域有第二套环境，则主节点名c2-m1，以此类推。

3. 监听端口规范：

   ​              统一主节点为10910，从节点为20910。

4. 从节点配置修改

   ![img](https://img-blog.csdnimg.cn/20190506094744430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NDM0NTcx,size_16,color_FFFFFF,t_70)

5. 备节点的brokerName填写所备的主节点的brokerName。

   ​     消息存放地址的最下层目录对应所备主节点的brokerName，改为c1-s2(节点1上的备节点连接的是节点2的)主节点。

   ​     BrokerRole为SLAVE

6. 日志目录修改,修改/opt/rocketmq-all-4.3.2-bin-release/conf目录下的3个日志配置文件，将三个配置文件中的所有${user.home}替换为/opt/rocketmq-all-4.3.2-bin-release,如下图所示：

   ![img](https://img-blog.csdnimg.cn/20190506095014200.png)

# 4.启停RocketMQ

1. 启动NAMESRV

   ~~~shell
   nohup sh /opt/rocketmq-all-4.3.2-bin-release/bin/mqnamesrv &
   ~~~

2. 启动主节点broker

   ~~~shell
   nohup   sh /opt/rocketmq-all-4.3.2-bin-release/bin/mqbroker -c /opt/rocketmq-all-4.3.2-bin-release/conf/2m-2s-async/broker-a.properties &
   ~~~

3. 启动从节点

   ~~~shell
   nohup sh /opt/rocketmq-all-4.3.2-bin-release/bin/mqbroker -c /opt/rocketmq-all-4.3.2-bin-release/conf/2m-2s-async/broker-a-s.properties &
   ~~~

4. 启动脚本

   启动命令在启动脚本/opt/rocketmq-all-4.3.2-bin-release/runmq.sh中可见，亦可执行该脚本启动。（原则上先启动主节点，再启动该主节点对应的备节点）

   ![img](https://img-blog.csdnimg.cn/20190506095120770.png)

5. 启动验证

   执行ps –ef|grep rocketmq查看相关进程状态，或使用内置命令jps查看均可

   ![img](https://img-blog.csdnimg.cn/20190506095153338.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NDM0NTcx,size_16,color_FFFFFF,t_70)

6. 停止RMQ

   使用kill命令查杀进程号即可，正常停止原则上先停备节点，再停对应的主节点

   --关闭nameserver和所有的broker:
      sh mqshutdown namesrv
      sh mqshutdown broker  注：避免使用kill -9，以免引起消息无法落盘的BUG

7. 控制台配置

   登陆RMQ控制台程序分区10.249.141.42,在/app目录下创建BASIC4(代表4.3版本)目录。 

   ![img](https://img-blog.csdnimg.cn/20190506095315264.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NDM0NTcx,size_16,color_FFFFFF,t_70)

8.  

   从其他目录下复制两个配置文件到该目录下

   ![img](https://img-blog.csdnimg.cn/20190506095343992.png)

    

    

9. 修改两个配置文件相关参数，如下图：

   ![img](https://img-blog.csdnimg.cn/20190506095359501.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NDM0NTcx,size_16,color_FFFFFF,t_70)

   

   ![img](https://img-blog.csdnimg.cn/20190506095408978.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NDM0NTcx,size_16,color_FFFFFF,t_70)

10. 启动控制台

    ~~~shell
    nohup /app/jdk1.8.0_77/bin/java -jar /app/jar/rocketmq-console-ng-3.0.0.jar --spring.config.location=/app/BASIC4/BASIC_application.properties &
    ~~~

11. 验证

    登陆10.249.141.42:8182打开控制台(端口号在配置文件中配置)

    注：园区网访问控制台需要打通到10.249.141.42的端口防火墙。跳板机可以直接访问。也可访问域名rmqmn.deppon.com。

12. 其他相关rocketmq常用命令：
    --查看命令用法
       sh mqadmin
       sh mqadmin help updateTopic
    --查看所有,新增，删除消费组group:
      sh mqadmin consumerProgress -n 192.168.186.10:9876
       sh mqadmin updateSubGroup -c rocketmq-cluster-a -n 192.168.186.10:9876 -g zkGroup
       sh mqadmin deleteSubGroup -c rocketmq-cluster-a -n 192.168.186.10:9876 -g zkGroup
    --查看指定消费组下的所有topic数据堆积情况：
      sh mqadmin consumerProgress -n 192.168.186.10:9876 -g zkGroup
    --删除topic，新增topic 删除topic
       sh mqadmin topicList -n 192.168.186.10:9876
       sh mqadmin updateTopic -c rocketmq-cluster-a -n 192.168.186.10:9876 -t zkTopic
      sh mqadmin deleteTopic –n 192.168.186.10:9876 –c rocketmq-cluster-a –t zkTopic

    --查看topic信息列表详情统计
      sh mqadmin topicstatus -n 192.168.186.10:9876 -t zkTopic
    --查询集群消息
       sh mqadmin clusterList -n 192.168.186.10:9876