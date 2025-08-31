
# 使用Nacos-Client获取配置


## 1.server启动

http://127.0.0.1:8848/nacos/

默认用户名和密码都是nacos

可以阅读快速开始的部分：
https://nacos.io/zh-cn/docs/quick-start.html

![截图](https://s3.ax1x.com/2021/01/11/s3ZyQg.png)

```java
public class NacosClient {

    public static void main(String[] args) throws NacosException {
        //服务器地址
        String serverAddr = "127.0.0.1:8848";
        ConfigService configService = NacosFactory.createConfigService(serverAddr);
        // 获取配置，获取的是整个配置文件对应的String，最后一个参数是超时时间
        String config = configService.getConfig
        ("test-data-id", "DEFAULT_GROUP", 5000);
        System.out.println(config);
    }
}
```
## 2.监听配置变更
```java
public class NacosClient {

    public static void main(String[] args) throws NacosException {
        String serverAddr = "127.0.0.1:8848";
        String dataId = "test-data-id";
        String groupId = "DEFAULT_GROUP";
        ConfigService configService = NacosFactory.createConfigService(serverAddr);
        final String config = configService.getConfig(dataId, groupId, 5000);
      //异步线程，监听事件变化
        configService.addListener(dataId, groupId, new AbstractConfigChangeListener() {
            @Override
            public void receiveConfigChange(ConfigChangeEvent event) {
            }

            @Override
            public void receiveConfigInfo(String configInfo) {
                super.receiveConfigInfo(configInfo);
                System.out.println("receive new ConfigInfo thread-->" + Thread.currentThread().getName());
                System.out.println("receive new ConfigInfo-->" + configInfo);
            }
        });
        while (true) {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
当内容变化时，会立即输出如下内容：
```xml
name=zhangshan
age=11
receive new ConfigInfo thread-->com.alibaba.nacos.client.Worker.longPolling.fixed-127.0.0.1_8848
receive new ConfigInfo-->name=zhangshan
age=12
```
**在本地启动可以到{user.home}/logs/nacos/config.log查看日志信息（包括错误日志）** 
SpringBoot2.4.1与nacos-config-spring-boot-starter0.2.7不兼容，缺少ConfigurationBeanFactoryMetadata这个类

springboot可以和spring cluoud是两种东西，二者并不能混为一谈。
比如springboot不支持bootstrap.yml配置
也不支持类似于@RefreshScope的注解
nacos的最新版不一定和springboot的最新版是匹配的。

#  

# nacos和Spring集成



nacos与spring的集成是最灵活的。因为不论是springboot还是spring cloud都又封装了一层配置

而且，springboot还是spring cloud面还涉及到版本兼容性的问题。

https://github.com/nacos-group/nacos-spring-project





## 3.nacos和spring boot的集成

1.nacos的最新版不一定和SpringBoot和SpringCloud的最新版是匹配的。比如SpringBoot2.4.1与nacos-config-spring-boot-starter0.2.7不兼容，缺少SpringBoot最新版2.41移除了ConfigurationBeanFactoryMetadata这个类。但是Nacos与Spring Boot的最新版nacos-config-spring-boot-starter0.2.7

2. SpringBoot可以和Spring Cluoud是两种东西，二者并不能混为一谈。Spring Cloud基于SpringBoot，但是对于实现机制不一样。

   比如SpringBoot不支持bootstrap.yml配置也不支持类似于@RefreshScope的注解。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>nacos</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- https://mvnrepository.com/artifact/com.alibaba.nacos/nacos-client -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>nacos-config-spring-boot-starter</artifactId>
            <version>0.2.7</version>
        </dependency>
    </dependencies>
</project>
```


server:
  port: 8088
spring:
  application:
    name: test-data-id
nacos:
  config:
    server-addr: localhost:8848



@SpringBootApplication
@NacosPropertySource(dataId = "test-data-id", autoRefreshed = true)
public class Application {


    /**
     * main方法
     *
     * @param args
     */
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}

@RestController
public class ConfigController {


    @NacosValue(value = "${name}", autoRefreshed = true)
    private String name;
    
    // @Value也同样生效 
    @NacosValue(value = "${age}", autoRefreshed = true)
    private int age;
    
    @GetMapping("/getConfig")
    @ResponseBody
    public String getConfig() {
        return name + ":" + age;
    }

}





nacos需要重写@value注解，以便支持。

nacos spring版本的依赖
<!--            <dependency>-->
<!--                <groupId>com.alibaba.nacos</groupId>-->
<!--                <artifactId>nacos-client</artifactId>-->
<!--                <version>1.4.0</version>-->
<!--            </dependency>-->

就会报错，因为nacos-client1.4版本的类在被移除了，使用就会抛出异常。
021-01-10 02:33:45.433 INFO [com.alibaba.nacos.client.Worker.longPolling.fixed-localhost_8848:c.a.n.c.c.i.CacheData] [fixed-localhost_8848] [notify-listener] time cost=1ms in ClientWorker, dataId=test-data-id, group=DEFAULT_GROUP, md5=4c9dd34d8097c56fa6fd63e7573b719b, listener=com.alibaba.nacos.spring.context.event.config.DelegatingEventPublishingListener@261cf99b 
2021-01-10 02:33:45.438 ERROR [NacosConfigListener-ThreadPool-1:c.a.n.c.c.i.CacheData] [fixed-localhost_8848] [notify-error] dataId=test-data-id, group=DEFAULT_GROUP, md5=4c9dd34d8097c56fa6fd63e7573b719b, listener=com.alibaba.nacos.spring.context.event.config.DelegatingEventPublishingListener@261cf99b tx={}
java.lang.ClassNotFoundException: com.alibaba.nacos.client.config.utils.MD5
	at java.net.URLClassLoader.findClass(URLClassLoader.java:382) ~[na:1.8.0_265]
	at java.lang.ClassLoader.loadClass(ClassLoader.java:418) ~[na:1.8.0_265]
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:352) ~[na:1.8.0_265]
	at java.lang.ClassLoader.loadClass(ClassLoader.java:351) ~[na:1.8.0_265]
	at com.alibaba.nacos.spring.context.annotation.config.NacosValueAnnotationBeanPostProcessor.onApplicationEvent(NacosValueAnnotationBeanPostProcessor.java:144) ~[nacos-spring-context-0.3.6.jar:na]
	at com.alibaba.nacos.spring.context.annotation.config.NacosValueAnnotationBeanPostProcessor.onApplicationEvent(NacosValueAnnotationBeanPostProcessor.java:55) ~[nacos-spring-context-0.3.6.jar:na]
	at org.springframework.context.event.SimpleApplicationEventMulticaster.doInvokeListener(SimpleApplicationEventMulticaster.java:172) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.event.SimpleApplicationEventMulticaster.invokeListener(SimpleApplicationEventMulticaster.java:165) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:139) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:404) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:361) ~[spring-context-5.2.8.RELEASE.jar:5.2.8.RELEASE]
	at com.alibaba.nacos.spring.context.event.DeferredApplicationEventPublisher.publishEvent(DeferredApplicationEventPublisher.java:60) ~[nacos-spring-context-0.3.6.jar:na]
	at com.alibaba.nacos.spring.context.event.config.DelegatingEventPublishingListener.publishEvent(DelegatingEventPublishingListener.java:98) ~[nacos-spring-context-0.3.6.jar:na]
	at com.alibaba.nacos.spring.context.event.config.DelegatingEventPublishingListener.receiveConfigInfo(DelegatingEventPublishingListener.java:92) ~[nacos-spring-context-0.3.6.jar:na]
	at com.alibaba.nacos.client.config.impl.CacheData$1.run(CacheData.java:210) ~[nacos-client-1.4.0.jar:na]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_265]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_265]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_265]

    因此，一定要使用对应版本的依赖。
    是需要 @NacosPropertySource 和 @NacosValue 都设置 autoRefreshed 为 true，设置之后，@NacosPropertySource 刷新的是整个配置源（PropertySource），@NacosValue 刷新的是单个字段，两个是有区别的。

另外，只有配置源刷新了，@NacosValue 目标只能才能被刷新。



# nacos with spring cloud

重写refreshscope



https://spring.io/projects/spring-cloud

springcloud 和spring boot的关系， 加上nacos目前支持的版本。springcloud采用Hoxton

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>nacos</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <spring.cloud-version>Hoxton.SR8</spring.cloud-version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud-version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- 由于只使用nacos，所以没有音乐别的依赖 -->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
                <version>2.2.3.RELEASE</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
    </dependencies>
</project>

在 Nacos Spring Cloud 中，dataId 的完整格式如下：

${prefix}-${spring.profiles.active}.${file-extension}
prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。
spring.profiles.active 即为当前环境对应的 profile，详情可以参考 Spring Boot文档。 注意：当 spring.profiles.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 ${prefix}.${file-extension}
file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。


​```xml
2021-01-10 14:58:53.308  INFO 13039 --- [           main] c.a.n.client.config.impl.ClientWorker    : [fixed-localhost_8848] [subscribe] test-data-id+DEFAULT_GROUP
2021-01-10 14:58:53.309  INFO 13039 --- [           main] c.a.nacos.client.config.impl.CacheData   : [fixed-localhost_8848] [add-listener] ok, tenant=, dataId=test-data-id, group=DEFAULT_GROUP, cnt=1
2021-01-10 14:58:53.310  INFO 13039 --- [           main] c.a.n.client.config.impl.ClientWorker    : [fixed-localhost_8848] [subscribe] test-data-id.yaml+DEFAULT_GROUP
2021-01-10 14:58:53.310  INFO 13039 --- [           main] c.a.nacos.client.config.impl.CacheData   : [fixed-localhost_8848] [add-listener] ok, tenant=, dataId=test-data-id.yaml, group=DEFAULT_GROUP, cnt=1
```




@SpringBootApplication
public class Application {


    /**
     * main方法
     *
     * @param args
     */
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}

注意在spring cloud环境中，使用@NacosValue(value = "${name}", autoRefreshed = true)并不会生效。

@RestController
@RefreshScope
public class ConfigController {

    @Value(value = "${name}")
    private String name;
    
    @Value(value = "${age}")
    private int age;
    
    @GetMapping("/getConfig")
    @ResponseBody
    public String getConfig() {
        return name + ":" + age;
    }

}
其他的获取配置的方式可以参考nacos的wiki：
https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config


server:
  port: 8088
spring:
  application:
    name: test-data-id
  cloud:
    nacos:
      config:
        server‐addr: localhost:8848 # 配置中心地址
        file‐extension: properties  # properties是naocs的默认文件格式，可以不用填写
        group: DEFAULT_GROUP # 默认组



https://blog.csdn.net/qq_31086797/article/details/107500079


  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency> 有点多余
        里面起作用的实际上是spring-cloud-context<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>nacos</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <properties>
        <spring.cloud-version>Hoxton.SR8</spring.cloud-version>
        <spring-boot.version>2.2.3.RELEASE</spring-boot.version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud-version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-config -->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
                <version>2.2.3.RELEASE</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-context</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
    </dependencies>
</project>
从源码可以看出来spring-cloud-context做了增强。增加了RefreshScope注解和bootstrap.yaml的文件解析





自定义注解

BeanPostProcessor ：BeanProcessor是对bean初始化的注解。
但有时候，注解并没有针对某个特定的bean。比如说是全局的配置bean。这个时候该怎么自定义注解呢？

很简单：扫描所有Spring的类，解析出注解
NacosPropertySourcePostProcessor




NacosConfigBootstrapConfiguration断点




@ConditionalOnMissingBean
该注解表示，如果存在它修饰的类的bean，则不需要再创建这个bean；

 

可以给该注解传入参数例如@ConditionOnMissingBean(name = "example")，

这个表示如果name为“example”的bean存在，这该注解修饰的代码块不执行。


https://baijiahao.baidu.com/s?id=1641004110489963465&wfr=spider&for=pc


nacos cloud模块这里的设计有点不好，所有的加载配置的过程全部在bean初始化完成，导致BeanFactoryPost想通过配置扩展无效。
只能自己通过扩展，将值塞到@Value可以识别的中间。

PropertySourceLocator 但这样做可能比较复杂。（需要考虑到对RefreshScope的影响）

