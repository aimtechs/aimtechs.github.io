---
published: true
title: "spring boot logging"
layout: post
author: jihwan, hwang
category: logback
tags:
- logback
---

# spring-boot logging

spring framework 는 apache commons logging 을 이용한다.

현재 대부분의 application 개발자는 slf4j 를 이용하여 logging 을 한다.

logger 구현체로는 log4j, log4j2, logback 등이 있다.

spring-boot-starter-xxx 를 maven or gradle 의존 설정을 하면, 의존 전이 특성으로 spring-boot-starter-logging 의존을 한다.

spring-boot-starter-logging 의존 설정을 보면, slf4j 기반, logback logger 를 정의하고 있다.

logging configuration xml 파일을 외부 디렉토리에서 관리 하고자 한다면, logging.config 옵션으로 지정하여 처리하자.

log4j2 logger를 이용하고 싶다면, 아래와 같이 의존 설정을 변경하면 된다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

# spring boot 1.5.1 loggers endpoint

spring-boot-starter-actuator 에는 많은 관리 기능과 모니터링 도구 들이 제공되고 있다.

1.5.1 부터는 JMX 와 MVC endpoint 를 이용하여, application logging level 을 운영중에 확인하고, 변경할 수 있다.

JMX 에서는 변경할 loggerName 와 logLevel 을 입력하면 되고,

MVC 에서는 POST 방식으로 /loggers/com.your.app 에 body 부분을 아래와 같이 json 으로 보내면 된다.

```json
{
  "configuredLevel": "DEBUG"
}
```
# slf4j

SLF4j 는 API, Binding, Bridge 3가지 모듈로 구성 되어 있다.

### API
  slf4j-api-${version}.jar


### Binding (반드시 하나만 정의 해야 함)  
  slf4j-${로거}-${version}.jar

  ex) slf4j-log4j12, slf4j-jdk14, slf4j-nop, slf4j-jcl, logback-classic

### Bridge

  ex) jcl-over-slf4j, log4j-over-slf4j, jul-to-slf4j

### 아래와 같은 순서로 레거시 로깅 api 가 slf4j 모듈을 통해 logger 구현체로 logging이 된다.

1. 레거시 로깅 코드 (commons-logging, log4j, jul)

2. Bridge

3. API

4. Binding

5. Logger 구현체


### logging tip

logging 을 json 형태로 남기도록 하자. 나중에 분석을 위한 조작이 편하다. (logstash 사용)

운영용 log 에서 standard output appender 는 무의미 하므로 제거하자.

서버 환경변수 [log 경로], application name 의 조합을 통해, 로그 경로를 만들자.
