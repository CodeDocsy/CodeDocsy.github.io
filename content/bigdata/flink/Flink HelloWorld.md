---
title: 快速启动Flink
order: 1
---

# 快速启动Flink

从https://flink.apache.org/downloads/ 下载最新的Flink，目前最新的是1.17.1（注意，不是下载源码）。

![2.1.png](/images/flink/2.1.png)

解压，运行bin目录下的`start-cluster.sh`文件（前提是安装好Java8，尽量安装Java8，Java11也可以，但是更高版本)。如果端口没有被占用，那么应该看到如下界面：

```
Starting cluster.
Starting standalonesession daemon on host bscoder.local.
Starting taskexecutor daemon on host bscoder.local.
```

进入到`http://localhost:8081/#/overview` 页面如下：

![image.png](/images/flink/2.2.png)

## 端口被占用时的处理

如果出现`Could not start rest endpoint on any port in port range 8081`这个错误就是说端口8081被占用了，。进入到log目录，查看日志，如果有如下错
通过`sudo lsof -i tcp:8081`命令查看端口被占用情况（注意，这个时候必须加sudo，否则什么都查不出来）：

如果端口被占用，进入到conf目录的 `flink-conf.yaml`文件， 对`rest.port` 取消注释，换成别的端口即可。
其他的端口类似。

## flink-conf.yaml配置说明

在1.17版本中，conf目录下有如下配置：flink-conf.yaml 配置、日志的配置文件、zk配置以及master/workers文件。

![image.png](/images/flink/2.3.png)

> 在flink1.14.0中已经移除sql-client-defaults.yml配置文件了。相关联的参考地址：https://issues.apache.org/jira/browse/FLINK-21454
> 以及https://cwiki.apache.org/confluence/display/FLINK/FLIP-163%3A+SQL+Client+Improvements
> 还是那句话，先不要管它。后

flink-conf.yaml的原文地址[在这里](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/deployment/config/)，**还是那句话，先只关注当下需要的**：

| 配置                     | 说明                                                                                    |
| ---------------------- | ------------------------------------------------------------------------------------- |
| jobmanager.rpc.address | Jobmanager的IP地址，即master地址。默认是localhost，此参数在HA环境下或者Yarn下无效，仅在local和无HA的standalone集群中有效 |
| jobmanager.rpc.port    | JobMamanger的通信端口，默认是6123，TaskManager以及各种client（如Sql Client）通过这个端口和JobMamanger通信       |
| rest.port              | Job Managener网页管理页面地址                                                                 |

## 运行官方的Sample

进入到Submit New Job，这个是上传jar包的地方：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f0c96e5ae914d67a393352b780ed936~tplv-k3u1fbpfcp-watermark.image?)

在解压的包中examples/streaming/中选择一个典型的流式处理的sample：SocketWindowWordCount.jar 并上传：
![企业微信截图_5d15e9e3-d8d6-4eb3-9ade-3a8b15de76ef.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44981677a0424e37a84eaa15bdda52bc~tplv-k3u1fbpfcp-watermark.image?)

点击这一行（注意不要点击到Delete上面）设置参数并且运行。

**注意，要先运行`nc -l 8888` 否则，启动任务会报错。因为任务启动以后，会建立socket连接，任务建立不上就会失败。**

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/736245fd9c59424aa089bf672dcead9f~tplv-k3u1fbpfcp-watermark.image?)

成功运行的截图如下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d7c6969c8f24c5f94d441f3513252c3~tplv-k3u1fbpfcp-watermark.image?)

这个时候，可能会号好奇很多点：

- parallelism和savepoint Path是什么？有什么用
- 为什么运行参数要这么填写？
- 各个页面的展示信息到底是什么含义？

是的，我们还是先忽略它。在下一章，我们将深入分析这些东西。现在，我们只需要确保能跑起来。

现在，可以在终端进行输入：

```java
bin ajian$ nc -l 8888
q
w
ww
w
ww
w
w
w
hello flink！
hello
```

在TaskManagers页面可以看到如下的输出：
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c4bd0304bb8429a88bafc478396f1bb~tplv-k3u1fbpfcp-watermark.image?)

至此，一个Flink的HelloWorld成功了！还是那句话，目标是Hello World，接下来，我们带着问题一步步的深入其中，探寻Flink的神秘地带
