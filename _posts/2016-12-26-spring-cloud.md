---
published: true
title: "spring cloud"
layout: post
author: jihwan, hwang
category: spring-cloud
tags:
- spring-cloud
---

# config server
설정은 프로세스 동작을 위한 필수 요소이다.
jvm option, properties, xml 파일 등에 많은 설정을 등록하여 application을 동작시킨다.
운영해야 할 application이 손으로 꼽을 정도의 갯수면 상관 없겠으나, 많은 수의 application을 운영 관리 하다 보면 설정 관리가 쉽지 않다.
실례로, 많은 application에 설정이 야기시키는 문제점으로는,
1. 각 application 마다 config 정보가 정해져 있고, application이 실행되는 server내 정해진 경로에 존재 하여, terminal 프로그램을 이용하여 이곳 저곳을 이동하며 확인했어야 했다.
2. 설정이 jar 파일내에 존재 한다면, 그 설정을 고치기 위해 빌드, 배포, 재기동을 해야 한다. 특히 재기동 부분에서는 부가적인 문제점으로 인하여 시스템 운영 노하우 및 관련 코드 작성이 필요 할 수도 있다.

결국, application의 기능 변경을 제외하고는 설정을 변경해야 하기 때문에, 서비스를 중단해야하는 상황이 발생되고 해결해야 한다.

수많은 설정 정보를 일괄 관리 하고, api 호출을 통해 application이 구동 되면 어떨까?

[12 factors](https://12factor.net/ko/) 에서는, 현대의 application이 어떻게 만들어 져야 하고, 운영 관리 되어야 하는지 등등 방법론을 제시한다.
본 내용은 spring cloud 에서 제공하는 기술을 이용하여 설정과 코드를 분리 하는 방법을 기술한다.
spring cloud 에서는 [config server](https://cloud.spring.io/spring-cloud-config/spring-cloud-config.html) 를 제공한다.

중요한 점은 config 정보는 source처럼 git or svn 에서 형상 관리 하도록 하고,
운영 시스템에서는 git or svn 에서 download 받아야 한다는 점이다.
한가지 특이한 점이 있다면, 일괄 관리 되는 파일이 commit 상태이어야 한다.
commit을 하지 않는다면, config server는 변경된 설정 정보를 반영하지 않는다.
또한, config 정보가 update되었다고 해서 application에서는 변경된 정보를 즉각 반영 하지 않는다.

config server 구성은 무척 쉽다. **@EnableConfigServer** 만 설정하면 된다.
그리고, 자신의 application.properties 파일에 아래와 같이 설정 한다. 아래는 git을 사용할 경우이다.

```bash
spring.cloud.config.server.git.uri= ${HOME}/yourConfigurationDirectory
server.port=8888
```

*${HOME}/yourConfigurationDirectory* 디렉토리 하위에 각 application 설정 정보가 존재 한다.
config server를 구동하고 [여기](http://localhost:8888/lot-service/development) 로 접속 해 보자.
이 내용은 *lot-service* application에 대한 설정 정보이다.

lot-service application 내용을 살펴 보자. spring에서 제공하는 profiles 기능을 이용하였으며, 아래 내용은 development profile 이다.

```bash
"server.port": "${PORT:8080}",
"server.contextPath": "/",
"spring.profiles": "development",
"spring.datasource.url": "jdbc:h2:mem:testDB",
"spring.datasource.username": "sa",
"spring.datasource.password": "",
"spring.datasource.initialize": true,
"message": "goodbye 2016 and hello 2017"
```

모든 설정 정보는 overriding 가능하다. 위에서는 ${PORT:8080} 으로 지정 해 보았다.
별다른 옵션 지정없이 구동하면, 톰캣 포트는 8080 이다. 포트를 변경하고자 하면, -DPORT=8181 이런 식으로 가능하다.

미리 만들어 둔 lot-service 자체는 자체 설정 정보를 더이상 관리 할 필요가 없고,
config server 정보만 알면 된다.
lot-service 에서 정의한 application.properties 내용을 모두 제거하고 아래와 같은 정보를 추가한다.

```bash
spring.cloud.config.uri=http://localhost:8888
spring.application.name=lot-service
```

마지막으로, application.properties 파일을 **bootstrap.properties** 로 이름을 변경한다.
이는 관례적으로 사용됨에 따라 변경을 해야 한다고 한다.


spring-cloud-config 는 application 재시작 없이 설정 정보를 변경하는 기능을 제공한다.
**@RefreshScope** annotation을 이용하여 적용한다.

```java
@RestController @RefreshScope
class MessageRestController {
private final String message;

public MessageRestController(@Value("${message}") String message) {
  this.message = message;
}

@GetMapping("/message")
String message() {
  return this.message;
}
}
```

config 정보 변경 및 commit 후 아래와 같은 message를 보내 보자.

```bash
curl -X POST http://localhost:8081/refresh
```
java -jar target/config-service-0.0.1-SNAPSHOT.jar


# eureka
MSA 구성에서 각 모듈의 lookup, scale status, health status 관리 모듈이다.
client에서 특정 모듈의 요청이 들어오면, eureka는 서비스가 어디에 있는지, 요청이 가능한 상태인지 체크를 해서
client에게 해당 모듈의 ip, port 정보를 돌려 주고, client는 해당 모듈에 요청하게 된다.

eureka는 서버와 클라이언트로 구성되고, 클라이언트는 자신의 정보를 서버에게,
서버는 클라이언트로 부터 받은 정보를 다른 클라이언트에게 전파하는 역활을 한다.
따라서, 특정 서비스에 많은 요청을 처리하기 위해, 서비스를 이루는 서버 또는 컨테이너가 늘어나는 경우,
늘어난 컨테이너들에서 동작하는 eureka 클라이언트들은 자신이 동작하는 순간,
서버에 자신의 정보를 전달하고 이 정보가 모든 클라이언트에 업데이트 된다.
결국 빠른 속도로 service scale in/out이 가능하다.

[eureka info](http://localhost:8761/eureka/apps) 정보들은 클라이언트가 부트될 때 서버로 보내지는
정보들이다.

eureka 서비스 자체는 각 application 인스턴스 정보만을 공유한다.

spring 에서 eureka server client 구성 역시 쉽다.

eureka server 코드

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServiceApplication.class, args);
	}
}
```

eureka-service configuration

```bash
# standalone mode
server.port=${PORT:8761}
eureka.client.register-with-eureka=true
eureka.client.fetch-registry=true
```

```bash
# replica mode

```

eureka server bootstrap.properties

```bash
spring.application.name=eureka-service
spring.cloud.config.uri=http://localhost:8888
```

eureka client 코드에는 모두 **@EnableDiscoveryClient** annotation 작성을 한다.


# Ribbon
client side loadbalancing 기능을 제공한다. 즉, 부하 분산을 해 준다.
(L4 switch, HAProxy 도 로드 밸런싱 기능을 제공하는데, 이는 server side loadbalancing 이다.)

server side loadbalancing의 경우, 아래와 같은 문제점이 있다.
1. switch 자체가 처리 할 수 있는 요청수의 한계가 존재. (그러나, 우리 도메인에서는 그다지...)
2. switch 설정의 어려움. 고비용. (가장 큰 문제점. 관리 포인트 증가)
3. switch 역시 장애를 대비하여 이중화 해야 함

위와 같은 문제점을 해결하고, 기존 loadbalancing의 역할을 쉽게 구현할 수 있는 것들 중 하나가 Ribbon 이다.

[여기까지 샘플 코드](https://github.com/aimtechs/microservices-demo)

[샘플 configuration](https://github.com/aimtechs/microservices-demo-config)

# Zuul

# Zipkin

# Hystrix Circuit Breakers
