### 一、Dubbo学习总结（一）--简单的使用流程与实例
  最近项目中使用了RPC远程服务调用框架，接下来总结一下dubbo的个人理解与使用。。。。
```word
其实，dubbo+zookeeper的使用大家可能听得多，但是具体干嘛用的，一头雾水，大家可以把dubbo理解成一个分布式框架，zk是管理dubbo服务的监控中心。具体如何，请看如下讲解。。。

一、Dubbo简介

1、Dubbo是什么?

Dubbo是阿里巴巴SOA服务化治理方案的核心框架，每天为2,000多个服务提供30多亿次访问量支持，并被广泛应用于阿里巴巴集团的各成员站点。
Dubbo是一个分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案，以及SOA服务治理方案。

2、核心部分

远程通讯:
    提供对多种基于长连接的NIO框架抽象封装，包括多种线程模型，序列化，以及“请求-响应”模式的信息交换方式

集群容错:
    提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持

自动发现:
    基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。

RPC能做什么？

透明化的远程方法调用，就像调用本地方法一样调用远程方法，只需简单配置，没有任何API侵入。软负载均衡及容错机制，可在内网替代F5等硬件负载均衡器，降低成本，减少单点。服务自动注册与发现，不再需要写死服务提供方地址，注册中心基于接口名查询服务提供者的IP地址，并且能够平滑添加或删除服务提供者

3、dubbo+ZK架构图



节点角色说明：
    Provider: 暴露服务的服务提供方。
    Consumer: 调用远程服务的服务消费方。
    Registry: 服务注册与发现的注册中心。
    Monitor:统计服务的调用次调和调用时间的监控中心。
    Container: 服务运行容器。

调用关系说明：
    0. 服务容器负责启动，加载，运行服务提供者。
    1.服务提供者在启动时，向注册中心注册自己提供的服务。
    2.服务消费者在启动时，向注册中心订阅自己所需的服务。
    3.注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
    4.服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
    5.服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

4、开发步骤

 步骤：

a.添加相关dubbo、zk依赖jar包

b.添加提供者(消费者)相关配置文件:

    b1.声明提供者(消费者)，

    b2.将dubbo协议、zk注册中心的服务地址暴露（消费者发现zk注册服务地址）

    b3.声明需要提供的服务接口与具体实现bean（需要远程调用的接口）
 
c.搭建zk服务监控中心

   c1.开启zk的服务，将dubbo的接口注册到服务中

   c2.启动dubbo的监控中心，监控提供者和消费者的接口调用和注册信息

二、入门实例

Dubbo采用全spring配置方式，透明化接入应用，对应用没有任何API侵入，只需用Spring加载Dubbo的配置即可，Dubbo基于Spring的Schema扩展进行加载。

新建maven工程，添加dubbo、spring、zk相关依赖

<properties> 
    <spring.version>4.3.7.RELEASE</spring.version> 
 </properties> 

<dependencies> 
    <!-- dubbo+zk相关 -->
    <dependency> 
      <groupId>com.alibaba</groupId>  
      <artifactId>dubbo</artifactId>  
      <version>2.5.3</version>  
      <exclusions> 
        <exclusion> 
          <groupId>org.springframework</groupId>  
          <artifactId>spring</artifactId> 
        </exclusion> 
      </exclusions> 
    </dependency>  
    <dependency> 
      <groupId>com.github.sgroschupf</groupId>  
      <artifactId>zkclient</artifactId>  
      <version>0.1</version> 
    </dependency>  
    <!-- spring相关 -->  
    <dependency> 
      <groupId>org.springframework</groupId>  
      <artifactId>spring-core</artifactId>  
      <version>${spring.version}</version> 
    </dependency>  
    <dependency> 
      <groupId>org.springframework</groupId>  
      <artifactId>spring-beans</artifactId>  
      <version>${spring.version}</version> 
    </dependency>  
    <dependency> 
      <groupId>org.springframework</groupId>  
      <artifactId>spring-context</artifactId>  
      <version>${spring.version}</version> 
    </dependency>  
    <dependency> 
      <groupId>org.springframework</groupId>  
      <artifactId>spring-jdbc</artifactId>  
      <version>${spring.version}</version> 
    </dependency>  
    <dependency> 
      <groupId>org.springframework</groupId>  
      <artifactId>spring-web</artifactId>  
      <version>${spring.version}</version> 
    </dependency>  
    <dependency> 
      <groupId>org.springframework</groupId>  
      <artifactId>spring-webmvc</artifactId>  
      <version>${spring.version}</version> 
    </dependency>  
    <dependency> 
      <groupId>org.springframework</groupId>  
      <artifactId>spring-aop</artifactId>  
      <version>${spring.version}</version> 
    </dependency>  
    <dependency> 
      <groupId>org.springframework</groupId>  
      <artifactId>spring-tx</artifactId>  
      <version>${spring.version}</version> 
    </dependency>  
    <dependency> 
      <groupId>org.springframework</groupId>  
      <artifactId>spring-orm</artifactId>  
      <version>${spring.version}</version> 
    </dependency>  
    <dependency> 
      <groupId>org.springframework</groupId>  
      <artifactId>spring-context-support</artifactId>  
      <version>${spring.version}</version> 
    </dependency>  
    <dependency> 
      <groupId>org.springframework</groupId>  
      <artifactId>spring-test</artifactId>  
      <version>${spring.version}</version> 
    </dependency>  
    <dependency> 
      <groupId>org.springframework</groupId>  
      <artifactId>spring-jms</artifactId>  
      <version>${spring.version}</version> 
    </dependency> 
  </dependencies> 

1、提供者

添加提供者配置application.xml，编写对应的接口与实现类。



测试启动：



2、消费者

添加消费者配置application.xml，远程调用需要的接口服务



测试消费者：



3、搭建好zk服务后,启动zk服务，进入dubbo的监控平台

dubbo+zk服务搭建：https://www.cnblogs.com/renhq/p/4654925.html

打开网址http://localhost:8090/dubbo-admin-2.5.4-SNAPSHOT/，输入root/root





dubbo相关配置：

https://www.cnblogs.com/linjiqin/p/5859153.html 

-------------------------------------------------------
Dubbo学习总结一

一、dubbo使用基本流程

新建服务接口
编写服务接口的实现类
注册服务的xml文件
编写启动服务测试类
新建消费Maven项目
创建相同路径相同的服务接口
消费服务的xml文件
编写启动消费服务的测试类
启动ZK，启动provider的测试类，向ZK注册服务
运行消费服务的测试类
二、dubbo使用，demo的具体实现

    1. 新建服务接口

      pom.xml引入一下jar包：

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.0</version>
        </dependency>
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.10</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>4.0.1</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.46</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.4</version>
        </dependency>
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.0.35.Final</version>
        </dependency>

     服务提供接口：

      public interface DemoProviderService {
          public String sayHello(String name);
      }

    2. 编写服务接口的实现类

      public class DemoProviderServiceImpl implements DemoProviderService {
          @Override
          public String sayHello(String name) {
               System.out.println("[" + new SimpleDateFormat("HH:mm:ss").format(new Date()) + "] Hello " + name + ", request from       consumer: " + RpcContext.getContext().getRemoteAddress());
               return "Hello " + name + ", response from provider: " + RpcContext.getContext().getLocalAddress();
          }
      }

    3. 注册服务的xml文件，写在src/main/resources文件夹下，名称 dubbo-demo-provider.xml

      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
             xmlns="http://www.springframework.org/schema/beans"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
             http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
          <!-- 服务提供者的应用名, 用来计算依赖关系 -->
          <dubbo:application name="demo-provider"/>
          <!-- 使用zookeeper注册中心暴露服务地址 -->
          <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
          <!-- 使用dubbo协议，在20880端口暴露服务 -->
          <dubbo:protocol name="dubbo" port="20880"/>
          <!-- 服务的实现类, 本地类 -->
          <bean id="demoProviderService" class="com.cslc.dubbo.service.impl.DemoProviderServiceImpl"/>
          <!-- 申明要暴露的服务接口 -->
          <dubbo:service interface="com.cslc.dubbo.service.DemoProviderService" ref="demoProviderService"/>
      </beans>

    4. 编写启动服务测试类

      public class Provider {
          public static void main(String[] args) {
              //从类路径下读取XML文件生成上下文环境
              ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("dubbo-demo-provider.xml");
              context.start();
              System.out.println("服务提供者向zookeeper注册中心注册服务成功（端口：20880）");
              try {
                  System.in.read();
              } catch (IOException e) {
                  // TODO Auto-generated catch block
                  e.printStackTrace();
              }
              context.close();
          }
      }

    5. 新建消费Maven项目

      pom.xml文件中引入jar包：
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.0</version>
        </dependency>
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.10</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>4.0.1</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.46</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.4</version>
        </dependency>
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.0.35.Final</version>
        </dependency>

    6. 创建相同路径相同的服务接口

      public interface DemoProviderService {
          public String sayHello(String name);
      }

    7. 消费服务的xml文件，写在src/main/resources文件夹下，名称 dubbo-demo-consumer.xml

      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
             xmlns="http://www.springframework.org/schema/beans"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
             http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
          <!-- 消费方应用名, 用来计算依赖关系，不要与提供方的名称一样 -->
          <dubbo:application name="demo-consumer"/>
          <!-- 使用zookeeper注册中心暴露服务地址 -->
          <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
          <!-- 使用dubbo协议，在20880端口暴露服务 -->
          <dubbo:protocol name="dubbo" port="20880"/>
          <!-- 生成远程服务代理，可以与本地的bean一样，check属性，启动的时候检查远程服务是否存在，一般设置为false，即使用的时候才去检查 -->
          <dubbo:reference id="demoProviderService" check="false" interface="com.cslc.dubbo.service.DemoProviderService"/>
      </beans>

    8. 编写启动消费服务的测试类

      public class Consumer {
          public static void main(String[] args) {
              ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("dubbo-demo-consumer.xml");
              context.start();
              DemoProviderService demoProviderService=(DemoProviderService) context.getBean("demoProviderService");
              String result=demoProviderService.sayHello("阿里");
              System.out.println("远程调用的结果："+result);
              try {
                  System.in.read();
              } catch (IOException e) {
                  // TODO Auto-generated catch block
                  e.printStackTrace();
              }
              context.close();
          }
      }

    9. 启动zookeeper，启动provider的测试类，向ZK注册服务

  10.运行消费服务的测试类

 

附：

  初学dubbo遇到的问题：

    1. Exception in thread "main" org.springframework.beans.factory.parsing.BeanDefinitionParsingException: Configuration problem: Unable to locate Spring NamespaceHandler for XML schema namespace [http://dubbo.apache.org/schema/dubbo]

Offending resource: class path resource [dubbo-demo-provider.xml]

......    at org.springframework.context.support.AbstractApplicationContext.obtainFreshBeanFactory(AbstractApplicationContext.java:614)
......

原因：阿里把项目贡献给了Apache 原头文件中

http://dubbo.apache.org/schema/dubbo与http://dubbo.apache.org/schema/dubbo/dubbo.xsd 已失效

配置文件头更换一下：

<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
       http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    2. Exception in thread "main" com.alibaba.dubbo.rpc.RpcException: No provider available from registry 127.0.0.1:2181 for service com.cslc.dubbo.DemoProviderService on consumer 172.20.29.235 use dubbo version 2.6.0, may be providers disabled or not registered ?
    ......
```
### 二、什么是多表关联查询，有几种多表关联查询的方式，分别是什么？
```word

什么是多表关联查询？
有时一个查询结果需要从两个或两个以上表中提取字段数据，此时需要使用的就是多表关联查询。
链接查询主要分为三种：内链接、外链接、交叉连接。
内链接
使用比较运算符（包括=、>、<、<>、>=、<=、!> 和!<）进行表间的比较操作，查询与连接条件相匹配的数据。根据所使用的比较方式不同，内连接分为等值连接、自然连接和自连接三种。
关键字：INNER JOIN
1.等值连接/相等链接：
使用”=“关系将表连接起来的查询，其查询结果中列出被链接表中的所有列，包括其中的重复列
2.自然链接
等值连接中去掉重复的列，形成的链接。
3.自链接
如果在一个连接查询中，涉及到的两个表是同一个表，这种查询称为自连接查询。
外链接
内连接只返回满足连接条件的数据行，外连接不只列出与连接条件相匹配的行，而是列出左表（左外连接时）、右表（右外连接时）或两个表（全外连接时）中所有符合搜索条件的数据行。外连接分为左外连接、右外链接、全外连接三种。
1.左外连接
关键字：LEFT[OUTER]JOIN
返回左表中的所有行，如果左表中行在右表中没有匹配行，则在相关联的结果集中右表的所有字段均为NULL。
2.右外连接
关键字：RIGHT[OUTER]JOIN
返回右表中的所有行，如果右表中行在左表中没有匹配行，则在左表中相关字段返回NULL值。
3.全外链接
关键字：FULL[OUTER]JOIN
返回两个连接中所有的记录数据，是左外链接和右外链接的并集。
交叉连接/笛卡尔积
关键字：CROSS JOIN
两个表做笛卡尔积，得到的结果集的行数是两个表中的行数的乘积。
注意：带有where条件的子句，往往会先生成两个表行数乘积的数据表，然后从根据where条件从中选择。
当数据量比价大的时候，笛卡尔积操作会很消耗数据库的性能

```

### springBoot与SSM框架相比有哪些不一样，有哪些优势？
```text
一、SSM优缺点应该分开来说的，比如
1）spring 不说了，核心ioc、aop技术，ioc解耦，使得代码复用，可维护性大幅度提升，aop提供切面编程，同样的增强了生产力。
2）spring mvc，是对比struts2等mvc框架来说的，不说struts2爆出的那么多安全漏洞，而且是类拦截，所有Action变量共享，同时是filter入口的，而spring mvc是方法拦截，controller独享request response数据，采用的serlvet入口，与spring无缝对接。开发而言，spring mvc更加轻量和低入门。
3）mybatis嘛，看业务场景，主要是mybatis的sql可以由开发者去掌控和调优，相对hibernate等orm框架来说，更加直观。在业务场景比较复杂，sql好多联合关联的情况下，mybatis谁用谁知道。当然缺点就是对sql不熟悉的开发者就不太友好了。
二、 SSM框架和spring boot全家桶相比有哪些优缺点？
这两者对比起来有点奇怪。因为SSM是WEB应用框架，涵盖整个应用层，而spring boot你可以看做一个启动、配置、快速开发的辅助框架，本身针对的是微服务。
springboot 只是为了提高开发效率，是为了提升生产力的：
1、springboot一个应用是一个可执行jar（启动类main方法启动web应用），而不像传统的war，内嵌tomcat容器，可以jar形式启动一个服务，可以快速部署发布web服务，微服务最好不过了。
2、将原有的xml配置，简化为java配置
3、当然结构可能跟一般的ssm有一定区别，但其实主要是在资源文件。
Spring Boot 默认“约定”从资源目录的这些子目录读取静态资源：
	• src/main/resources/META-INF/resources
	• src/main/resources/static （推荐）
	• src/main/resources/public

来自 <https://www.cnblogs.com/mqflive81/p/10793217.html> 

--------------------------------------------------------------------

一、SSM优缺点应该分开来说的，比如
1）spring 不说了，核心ioc、aop技术，ioc解耦，使得代码复用，可维护性大幅度提升，aop提供切面编程，同样的增强了生产力。
2）spring mvc，是对比struts2等mvc框架来说的，不说struts2爆出的那么多安全漏洞，而且是类拦截，所有Action变量共享，同时是filter入口的，而spring mvc是方法拦截，controller独享request response数据，采用的serlvet入口，与spring无缝对接。开发而言，spring mvc更加轻量和低入门。
3）mybatis嘛，看业务场景，主要是mybatis的sql可以由开发者去掌控和调优，相对hibernate等orm框架来说，更加直观。在业务场景比较复杂，sql好多联合关联的情况下，mybatis谁用谁知道。当然缺点就是对sql不熟悉的开发者就不太友好了。
二、 SSM框架和spring boot全家桶相比有哪些优缺点？
这两者对比起来有点奇怪。因为SSM是WEB应用框架，涵盖整个应用层，而spring boot你可以看做一个启动、配置、快速开发的辅助框架，本身针对的是微服务。
springboot 只是为了提高开发效率，是为了提升生产力的：
1、springboot一个应用是一个可执行jar（启动类main方法启动web应用），而不像传统的war，内嵌tomcat容器，可以jar形式启动一个服务，可以快速部署发布web服务，微服务最好不过了。
2、将原有的xml配置，简化为java配置
3、当然结构可能跟一般的ssm有一定区别，但其实主要是在资源文件。
Spring Boot 默认“约定”从资源目录的这些子目录读取静态资源：
	• src/main/resources/META-INF/resources
	• src/main/resources/static （推荐）
	• src/main/resources/public

```
