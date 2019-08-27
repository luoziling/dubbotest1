# springboot整合dubbo+zookeeper

项目结构：

![01](G:\学习\csdn\pic\2019.8.27\01.png)

**注意：在做此demo之前应该先自行搭建好zookeeper环境**

**也可以搭建dubbo监控环境**

## 1.随便创建一个项目，之后添加一个maven项目用于存放接口

```java
package com.wzb.service;

/**
 * @author Satsuki
 * @time 2019/8/27 17:51
 * @description:
 * 一些模拟数据库事务的方法
 */
public interface TestService {
    public void ins();
    public void del();
    public void upd();
    public void sel();
}
```

## 2.创捷服务提供者（将服务注册到zookeeper）

pom文件：

```pom
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.7.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.wzb</groupId>
    <artifactId>provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>provider</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.wzb</groupId>
            <artifactId>service</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.3</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.7.3</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>4.0.1</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.8.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.13</version>
            <type>pom</type>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.10</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

application.properties:

```properties
#spring项目名
spring.application.name=dubbo_auto_configuration_provider_demo
#Dubbo provider configuration
dubbo.application.name=dubbo_provider
dubbo.registry.protocol=zookeeper
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.protocol.name=dubbo
dubbo.protocol.port=20880
#扫描注解包通过该设置将服务注册到zookeeper
dubbo.scan.base-packages=com.wzb.provider.service
```

具体服务实现：

```java
package com.wzb.provider.service.impl;

import com.wzb.service.TestService;
import org.apache.dubbo.config.annotation.Service;



/**
 * @author Satsuki
 * @time 2019/8/27 15:35
 * @description:
 * 模拟数据库事务实现
 */
@Service(version = "1.0.0",interfaceClass = TestService.class)
//@Service(version = "1.0.0")
public class TestServiceImpl implements TestService {
    @Override
    public void ins() {
        System.out.println("insert");
    }

    @Override
    public void del() {
        System.out.println("delete");
    }

    @Override
    public void upd() {
        System.out.println("update");
    }

    @Override
    public void sel() {
        System.out.println("select");
    }
}
```

启动类：

```java
package com.wzb.provider;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }

}
```

### 步骤总结:

1.搭建pom环境

2.写好配置文件（一定要写上dubbo.scan.base-packages该配置会扫描你写的服务并注册到zookeeper）

3.具体服务实现

4.运行springboot启动类即可

## 3.创建服务消费者（服务使用者：通过远程调用服务使用）

pom文件与provider一致不再列出

application.properties:

```properties
#dubbo configuration
dubbo.application.name=dubbo_consumer
dubbo.registry.protocol=zookeeper
dubbo.registry.address=zookeeper://127.0.0.1:2181

#避免端口冲突
server.port=8085
```

服务调用：

```java
package com.wzb.consumer.controller;


import com.wzb.service.TestService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


import javax.annotation.Resource;


/**
 * @author Satsuki
 * @time 2019/8/27 15:58
 * @description:
 * 调用dubbo
 */
@RestController
public class TestController {

//    @Resource
//    private TestService testServiceImpl;
    @Reference(version = "1.0.0")
    private TestService testServiceImpl;

    @RequestMapping("/ins")
    public String ins(){
        testServiceImpl.ins();
        return "ins";
    }

    @RequestMapping("/del")
    public String del(){
        testServiceImpl.del();
        return "del";
    }

    @RequestMapping("/upd")
    public String upd(){
        testServiceImpl.upd();
        return "upd";
    }

    @RequestMapping("/sel")
    public String sel(){
        testServiceImpl.sel();
        return "sel";
    }
}
```

之后只要写一个springboot启动类启动服务并且通过url访问即可测试调用服务

结果如下：

![02](G:\学习\csdn\pic\2019.8.27\02.png)

![03](G:\学习\csdn\pic\2019.8.27\03.png)

完成



## 踩坑：

java.lang.NoClassDefFoundError: org/apache/curator/framework/recipes/cache/TreeCacheListener

在pom文件中加入这两个依赖即可

```pom
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.0.1</version>
</dependency>

<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>2.8.0</version>
</dependency>
```
