## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/chx516/chxspring.github.io/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.
  # Springboot+SpringCloud+Eureka整合

## 1. Eureka核心原理

### 1.1 基本原理

> Eureks Service  服务提供者
>
> Eureka Client 服务消费者

```tex
服务启动后向Eureka注册，Eureka Server会将注册信息向其他Eureka Server进行同步，当服务消费者要调用服务提供者，则向服务注册中心获取服务提供者地址，然后会将服务提供者地址缓存在本地，下次再调用时，则直接从本地缓存中取，完成一次调用。
```

### 1.2 Eureka自我保护机制

```tex
在默认配置中EurekaServer服务在一定时间（默认为90秒）没接受到某个服务的心跳连接后，EurekaServer会注销该服务。但是会存在当网络分区发生故障，导致该时间内没有心跳连接，但该服务本身还是健康运行的情况。Eureka通过“自我保护模式”来解决这个问题。
在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。
```

### 1.3 Eureka与Zookeeper的区别

```tex
CAP理论指出，一个分布式系统不可能同时满足C(一致性)、A(可用性)和P(分区容错性)。由于分区容错性P在是分布式系统中必须要保证的，因此我们只能在A和C之间进行权衡。
Zookeeper保证CP
Zookeeper 为主从结构，有leader节点和follow节点。当leader节点down掉之后，剩余节点会重新进行选举。选举过程中会导致服务不可用，丢掉了可用行。
Eureka保证CP
Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册或时如果发现连接失败，则会自动切换至其它节点，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的信息可能不是最新的(不保证强一致性)。
```

## 2. 提供服务者  Eureka service

### 2.1 引入maven 依赖

```xml
  <!--eureka-server服务端 -->
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-eureka-server</artifactId>
   </dependency>
```

### 2.2 配置yml文件

```yml
# application.yml
server:
	port:7000  #springboot端口，可随自行情况设置
eureka:
	instance:
    	hostname: localhost  #eureka服务端的实例名字
  	client:
    	register-with-eureka: false    #表识不向注册中心注册自己
    	fetch-registry: false   #表示自己就是注册中心，职责是维护服务实例，并不需要去检索服务
     	service-url:
      		defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/   
            #设置与eureka server交互的地址查询服务和注册服务都需要依赖这个地址
```

### 2.3 配置启动类---@EnableEurekaServer启动Eureka service

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer //启动Eureka service
public class DomEurekaService7000Application {
    public static void main(String[] args) {
        SpringApplication.run(DomEurekaService7000Application.class, args);
    }
}
```

## 3. 服务消费者  Eureka Client

### 3.1 引入maven依赖

```xml
  <!--eureka-client客户端 -->
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-eureka-client</artifactId>
   </dependency>
```

### 3.2 配置yml文件

```yml
# application.yml
server:
	port:8001  #springboot端口，可随自行情况设置
spring:
  application:
    name: dom-client #重点，此处对后面服务影响关键
eureka
  	client:
    	register-with-eureka: true    
    	fetch-registry: true   
     	service-url:
      		defaultZone: http://localhost:7000/eureka/   		#配置eureka service 地址
```

### 3.3  配置启动类---@EnableEurekaClient启动Eureka Client

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClien //启动Eureka Client
public class DomEurekaService7000Application {
    public static void main(String[] args) {
        SpringApplication.run(DomEurekaService7000Application.class, args);
    }
}
```

