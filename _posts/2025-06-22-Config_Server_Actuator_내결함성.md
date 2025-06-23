---
layout: post
title: "Config Server, Actuator, 내결함성"
date: 2025-06-21 22:00:00 +0900
categories: msa
background: '/img/posts/12.jpg'
---


-   좋은 MSA 구조의 Application을 개발하려면?
    
    # 좋은 MSA 구조의 Application을 개발하려면?
    
    <aside> 💡
    
    좋은 MSA 구조의 애플리케이션 개발은 REST API 설계, 게이트웨이 설정, 구성 외부화, 모니터링, 내결함성 패턴, 로깅 및 분산 추적, CI/CD 등 다양한 영역의 모범 사례와 도구를 종합적으로 활용하는 것이 핵심이다. 각 항목은 독립적이면서도 상호 보완적인 역할을 수행하며, 전체 시스템의 유연성과 확장성을 보장한다. 이를 통해 안정적이고 확장 가능한 서비스 환경을 구축할 수 있다.
    
    </aside>
    
    ## 1. REST API 설계
    
    -   **표준화된 통신 방식**
        
        REST API는 마이크로서비스 간 통신의 표준화된 인터페이스를 제공한다. API 명세를 문서화하기 위해 Swagger 등의 도구를 활용하면, 협업 및 유지보수가 용이해진다.
        
    -   **자원 중심 설계**
        
        URI 설계와 HTTP 메서드(GET, POST, PUT, DELETE 등)를 적절하게 활용하여 자원 중심의 설계를 구현한다. 클라이언트와 서버 간의 명확한 역할 분리가 이루어져야 한다.
        
    -   **버전 관리**
        
        API의 변경에 따른 호환성을 유지하기 위해 버전 관리 전략을 수립한다. 예를 들어, URI에 버전 정보를 포함하는 방식(versioning)을 도입할 수 있다.
        
    
    ## 2. Gateway 설정
    
    -   **보안 및 추상화**
        
        외부에 내부 인스턴스를 직접 노출하지 않고, API 게이트웨이를 통해 요청을 라우팅한다. 이를 통해 보안성을 강화하고, 내부 서비스 구조를 추상화한다.
        
    -   **로드 밸런싱 및 인증**
        
        API 게이트웨이는 로드 밸런싱, 인증/인가, 요청 라우팅 등의 기능을 제공한다. Spring Cloud Gateway, Netflix Zuul 등 다양한 도구를 활용할 수 있다.
        
    -   **스케일 아웃**
        
        게이트웨이를 통한 분산 처리와 요청 관리로 내부 서비스의 스케일 아웃을 효과적으로 지원할 수 있다.
        
    
    ## 3. 구성 외부화
    
    -   **Spring Cloud Config Server**
        
        Spring 기반 애플리케이션의 환경별 구성 정보를 중앙에서 관리하기 위한 대표적인 도구이다. 구성 외부화를 통해 애플리케이션 재배포 없이도 설정을 변경할 수 있다.
        
    -   **추가 대안 기술**
        
        -   **Consul:** 서비스 디스커버리와 분산 키-값 저장소 기능을 함께 제공하여, 구성 관리와 헬스 체크를 동시에 지원한다.
        -   **Kubernetes ConfigMaps 및 Secrets:** 컨테이너 환경에서 애플리케이션 설정을 네이티브 방식으로 관리할 수 있으며, 환경 변수와 설정 파일 등을 외부화할 때 유용하다.
        -   **클라우드 기반 구성 관리 서비스**
            -   AWS Systems Manager Parameter Store, AWS AppConfig와 같은 서비스는 클라우드 환경에서 안전하게 구성 데이터를 저장하고 동적으로 반영할 수 있도록 지원한다.
            -   HashiCorp Vault는 주로 비밀 관리에 초점을 두지만, 동적 구성 관리 기능도 제공하여 보안에 민감한 환경에서 활용된다.
    
    ## 4. 모니터링
    
    -   **데이터 수집**
        
        Spring Actuator를 활용하여 애플리케이션의 상태(health check, 메트릭 등)를 실시간으로 노출한다. Prometheus는 시간에 따른 메트릭 데이터를 수집하며, 서버의 CPU 사용률, 메모리, 네트워크 트래픽 등 다양한 지표를 모니터링한다.
        
    -   **데이터 통합 및 시각화**
        
        Prometheus로 수집한 데이터를 Grafana를 통해 대시보드, 차트, 그래프 형태로 시각화하여 한눈에 시스템 상태를 파악할 수 있게 한다. 이는 장애 발생 시 빠른 대응과 성능 분석에 중요한 역할을 한다.
        
    -   **경고 시스템**
        
        수집된 데이터를 기반으로 임계치 초과나 오류 발생 시 자동 알림을 제공하는 경고 시스템을 도입하여, 빠른 문제 인식과 해결을 지원한다.
        
    
    ## 5. 내결함성 (Resilience) 패턴
    
    -   **Circuit Breaker**
        
        Netflix Hystrix 또는 최신 대안인 Resilience4j를 활용하여, 서비스 간 호출에서 장애가 발생할 경우 자동으로 회로를 차단하여 장애 전파를 방지한다. 이로써 한 서비스의 장애가 전체 시스템에 영향을 미치는 것을 예방할 수 있다.
        
    -   **Fallback Mechanism**
        
        서비스 장애 시 대체 로직을 수행하여 기본 응답이나 캐시된 데이터를 제공함으로써, 시스템의 연속성을 보장한다. 이를 통해 일부 서비스 장애가 발생하더라도 전체 사용자 경험에 미치는 영향을 최소화할 수 있다.
        
    
    ## 6. 로깅 및 분산 추적
    
    -   **중앙 집중식 로그 관리**
        
        로그를 중앙에서 집계하고 분석하는 시스템을 구축하여, 각 서비스에서 발생하는 로그를 통합적으로 관리한다. 이는 장애 원인 파악과 성능 모니터링에 필수적이다.
        
    -   **컨텍스트 전파**
        
        각 서비스에 trace-id, span-id 등을 포함하여 분산 시스템 전반의 호출 관계와 성능, 오류 상황을 추적한다. 이를 통해 전체 트랜잭션의 진행상황을 정확하게 분석할 수 있다.
        
    -   **Zipkin**
        
        -   Spring Cloud 환경에서 분산 시스템 내 호출 관계와 성능을 추적하며, 문제 발생 시 원인 분석을 지원한다.
    -   **추가 대안 기술**
        
        -   **Jaeger:** 오픈소스 분산 추적 시스템으로, 트랜잭션의 흐름, 서비스 간 의존성, 지연 시간 등을 시각화하여 분석할 수 있다.
        -   **OpenTelemetry:** 다양한 언어와 프레임워크에서 트레이싱 데이터를 표준화하여 수집하고, Jaeger, Zipkin, Elastic APM 등 여러 백엔드 시스템과 연동 가능하다.
        -   **클라우드 기반 서비스**
            -   **AWS CloudWatch 및 AWS X-Ray**
                
                AWS 환경에서 CloudWatch Logs와 Metrics를 사용해 로그를 중앙에서 관리하고, X-Ray를 통해 분산 추적 기능을 제공하여 서비스 간 호출 관계와 지연, 오류를 분석할 수 있다.
                
    
    ## 7. CI/CD (지속적 통합 및 지속적 배포) → DevOps 에서 학습 예정
    
    -   **자동화 파이프라인**
        
        Jenkins, GitLab CI, CircleCI, GitHub Actions 등 다양한 도구를 활용하여 코드 빌드, 테스트, 배포 과정을 자동화한다. 이는 코드 품질을 높이고 배포 주기를 단축하는 데 기여한다.
        
    -   **테스트와 검증**
        
        지속적 통합(CI) 과정에서 단위 테스트, 통합 테스트, E2E 테스트 등을 수행하여 애플리케이션의 안정성을 확보한다. 지속적 배포(CD)를 통해 변경 사항을 빠르게 프로덕션에 반영한다.
        
    -   **롤백 및 모니터링**
        
        배포 과정에서 문제가 발생할 경우 롤백 전략을 마련하고, 배포 후 모니터링을 통해 안정성을 지속적으로 검증한다.
        

# 4. Config Server, Actuator, 내결함성

## 4-1. Spring Cloud Config Server

### 4-1-1. MSA 환경에서의 구성 관리

-   배포 되는 실제 코드와 구성 정보를 완전히 분리한다.
-   여러 환경에서도 절대 변경 되지 않는 불변(immutable) 애플리케이션 이미지를 빌드한다.
-   서버가 시작할 때 마이크로서비스가 읽어 오는 환경 변수 또는 중앙 저장소를 통해 모든 애플리케이션 구성 정보를 주입한다.

### 4-1-2. Spring Cloud Config Server

![[https://pankajtechblogs.dev/spring-cloud-config-server-auto-reload-config-properties-zero-touch-63c4423beb59](https://pankajtechblogs.dev/spring-cloud-config-server-auto-reload-config-properties-zero-touch-63c4423beb59)

다양한 구성 관리 솔루션 중 스프링 부트와 밀접하게 통합 되어 있어 사용하기 쉬우며 깃 소스 제어 플랫폼 또는 하시코프 볼트와 바로 통합할 수 있다.]([https://prod-files-secure.s3.us-west-2.amazonaws.com/2ed584e0-2b03-4eac-8d4b-4490d601f1ba/17da7a70-6392-47cf-8043-d74e7e11be82/1_5fNSuIhy02ix4mKEdWLTrA.webp](https://prod-files-secure.s3.us-west-2.amazonaws.com/2ed584e0-2b03-4eac-8d4b-4490d601f1ba/17da7a70-6392-47cf-8043-d74e7e11be82/1_5fNSuIhy02ix4mKEdWLTrA.webp))

[https://pankajtechblogs.dev/spring-cloud-config-server-auto-reload-config-properties-zero-touch-63c4423beb59](https://pankajtechblogs.dev/spring-cloud-config-server-auto-reload-config-properties-zero-touch-63c4423beb59)

다양한 구성 관리 솔루션 중 스프링 부트와 밀접하게 통합 되어 있어 사용하기 쉬우며 깃 소스 제어 플랫폼 또는 하시코프 볼트와 바로 통합할 수 있다.

### 주요 기능

<aside> 💡

-   중앙 집중식 구성 관리: 여러 환경과 애플리케이션에 대한 구성을 한 곳에서 관리할 수 있다.
-   동적 구성 업데이트: 런타임 중에 구성을 변경하고 클라이언트에 즉시 반영할 수 있다.
-   버전 관리: Git 등의 버전 관리 시스템과 통합하여 구성 변경 이력을 추적할 수 있다.
-   보안: 중요한 구성 정보를 암호화하여 저장하고 전송할 수 있다. </aside>

### 4-1-4. 설정 방법

### Spring Cloud Config Server측 설정

Spring Colud Config Server dependency를 추가한다.

```groovy
implementation 'org.springframework.cloud:spring-cloud-config-server'

```

`@EnableConfigServer`는 Spring Cloud Config Server를 활성화하는 애노테이션이다. 이 애노테이션을 사용하면 Spring Boot 애플리케이션이 구성 서버로 동작하게 된다.

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}

```

`application.yml` 파일에 spring.cloud.config.server.git.uri 설정에 읽어올 설정파일의 경로를 지정한다.

-   Local Repository인 경우 Local Repository의 경로가 기입 되면 된다.
-   Github Remote Repository인 경우 Github Repository의 주소가 기입 되면 된다.
    -   public한 경우 주소만 기입해도 된다.
    -   private한 경우 username, password가 같이 기입 되어야 한다.
    -   username, password 노출을 피하고 싶을 경우 SSH key 생성 후 github에 공개키, config server에 개인키를 추가하여 설정하는 것도 가능하다.

```yaml
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: [설정파일이 담긴 깃 레포지토리 경로]

```

### Service Instance측 설정

bootstrap.yml 사용을 위한 dependency를 추가한다.

```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-config'
implementation 'org.springframework.cloud:spring-cloud-starter-bootstrap'

```

-   [bootstrap.properties](http://bootstrap.properties) or bootstrap.yml은 스프링 클라우드의 특정 파일 타입으로 [application.properties](http://application.properties) or application.yml 파일을 사용하는 컴포넌트보다 ApplicationContext로 먼저 로드 된다.
-   스프링 애플리케이션 이름, **스프링 클라우드 구성 서버 위치**, 암호화/복호화 정보 등을 지정한다.
-   bootstrap.yml 을 작성한다.

```yaml
spring:
  cloud:
    config:
      uri: [스프링 클라우드 컨피그 서버 주소]
      name: [읽어올 파일]

```

-   application.yml 에서 설정파일이 분리 된 내용을 제거하고 bootstrap.yml 파일을 읽어올 수 있도록 import 한다.

```yaml
spring:
  config:
    import:
      - classpath:/bootstrap.yml

```

## 4-2. Spring Actuator

### 4-2-1. Spring Actuator란?

<aside> 💡

Spring Actuator는 Spring Boot 애플리케이션의 운영 환경에서 모니터링 및 관리를 위한 기능을 제공하는 라이브러리이다.

</aside>

-   애플리케이션 상태, 메트릭, 환경 설정 등의 정보를 실시간으로 확인할 수 있다.
-   HTTP 엔드포인트나 JMX를 통해 애플리케이션의 내부 상태를 모니터링하고 관리할 수 있다.
-   애플리케이션의 건강 상태 체크, 로그 레벨 변경, 스레드 덤프 확인 등 다양한 운영 기능을 제공한다.
-   필요에 따라 사용자 정의 엔드포인트를 추가하여 기능을 확장할 수 있다.

### 4-2-2. 설정 방법

actuator 사용을 위한 dependency를 추가한다.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator'

```

액추에이터의 모든 엔드포인트의 경로에는 /actuator가 앞에 붙는다.

```yaml
### 기본 경로 수정
management:
  endpoints:
    web:
      base-path: /management

```

대부분의 액추에이터 엔드포인트는 민감한 정보를 제공하므로 보안 처리가 되어야 한다. 스프링 시큐리티를 사용해서 액추에이터를 보안 처리할 수 있다. 그러나 액추에이터 자체로는 보안 처리가 되어 있지 않으므로 대부분의 엔드포인트가 기본적으로 비활성화되어 있다.

```yaml
### 엔드포인트의 노출 여부 제어
management:
  endpoints:
    web:
      exposure:
        include: health,info,beans,conditions 
        exclude: threaddump, heapdump

```

### 4-2-3. Actuator Endpoint 종류

엔드포인트의 종류는 굉장히 많으므로 모든 종류는 공식 문서를 통해서 확인하도록 한다. 또한 기능에 따라 dependency 추가 및 bean 등록이 필요 할 수 있으니 확인 후 활용한다.

[Endpoints :: Spring Boot](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html#actuator.endpoints)

몇 가지 대표적인 엔드포인트는 다음과 같다.

| 엔드포인트 | 설명 |
| --- | --- |
| `/actuator/health` | 애플리케이션의 헬스 체크 |
| `/actuator/metrics` | 애플리케이션의 메트릭 정보 |
| `/actuator/info` | 애플리케이션 메타데이터 |
| `/actuator/env` | 환경 변수 및 설정 |
| `/actuator/loggers` | 로그 레벨 관리 |
| `/actuator/refresh` | 애플리케이션의 환경 설정 리프레시 |
| `/actuator/threaddump` | 스레드 덤프 정보 |
| `/actuator/beans` | Spring Bean 정보 |
| `/actuator/httpexchanges` | HTTP 교환 정보 |

1.  **GET /actuator/health**
    
    ```json
    {
      "status": "UP"
    }
    
    ```
    
2.  **GET /actuator/metrics**
    
    ```json
    {
      "names": ["jvm.memory.used", "jvm.gc.pause", ...]
    }
    
    
    ```
    
3.  **GET /actuator/info**
    
    ```json
    {
      "app": {
        "name": "MyApp",
        "version": "1.0.0"
      }
    }
    
    
    ```
    
4.  **GET /actuator/env**
    
    ```json
    {
      "activeProfiles": [],
      "propertySources": [...]
    }
    
    
    ```
    
5.  **GET /actuator/loggers**
    
    ```json
    {
      "levels": {
        "ROOT": "INFO",
        "com.example": "DEBUG"
      }
    }
    
    
    ```
    
6.  **GET /actuator/threaddump**
    
    ```json
    {
      "threads": [
        {
          "name": "main",
          "state": "RUNNABLE",
          ...
        }
      ]
    }
    
    
    ```
    
7.  **GET /actuator/beans**
    
    ```json
    {
      "contexts": {
        "application": {
          "beanDefinitionCount": 50,
          ...
        }
      }
    }
    
    
    ```
    
8.  **GET /actuator/httpexchanges**
    
    ```json
    {
      "exchanges": [
        {
          "timestamp": "2024-09-26T12:00:00Z",
          "method": "GET",
          "uri": "/api/example",
          ...
        }
      ]
    }
    
    
    ```
    

## 4-3. 내결함성 (Resilience) 패턴

<aside> 💡

내결함성 패턴은 한 서비스의 장애가 전체 시스템에 영향을 미치지 않도록 분리하고, 장애 상황에서도 서비스의 연속성을 보장하기 위해 핵심적으로 적용된다.

</aside>

### 4-3-1. Circuit Breaker와 Fallback 메커니즘

-   **핵심 개념**
    -   **Circuit Breaker:** 시스템 내 특정 서비스 호출 시 장애가 발생하면 일정 기간 동안 호출을 차단하여 장애 전파를 막는 패턴이다. Netflix Hystrix, Resilience4j 등이 대표적인 구현체이다.
    -   **Fallback Mechanism:** 장애 발생 시 대체 로직이나 기본 응답, 캐시된 데이터를 제공하여 사용자에게 최소한의 서비스를 유지해 준다.
-   **주요 이점**
    -   서비스 장애에 따른 전체 시스템 영향 최소화
    -   빠른 장애 탐지 및 자동 복구
    -   안정적인 사용자 경험 제공

### 4-3-2. Gateway에서 서비스 장애 시 내결함성 처리

API Gateway (예: Spring Cloud Gateway)를 통해 외부 클라이언트로부터 들어오는 요청을 내부 서비스(예: 사용자 서비스)로 라우팅할 때, 사용자 서비스에 장애가 발생하면 미리 정의된 fallback URI로 요청을 전달하는 방식이다.

-   **구성 요소 및 동작 원리**
    -   **Gateway Route 설정**
        -   `lb://SWCAMP-USER-SERVICE`를 대상으로 라우팅하며, 로드밸런싱을 수행
        -   요청 경로를 재작성(RewritePath)하여 내부 API 주소와 맞추고, Circuit Breaker 필터를 적용
        -   Circuit Breaker 이름(`userServiceCB`)가 설정되어 있고, 장애 발생 시 `fallbackUri: forward:/fallback/user`로 포워딩되어 대체 응답을 제공
    -   **Fallback Controller**
        -   `/fallback/user` 엔드포인트를 구현한 컨트롤러가 정의되어 있으며, 사용자에게 장애 상황에 대한 메시지를 반환하여 서비스 불능 상태를 최소화

### 4-3-3. Order Service에서 호출하는 User Service 장애시 내결함성 처리

Order Service 내에서 사용자 서비스(예: 사용자 등급 조회)를 호출할 때 Resilience4j의 Circuit Breaker를 적용하여, user service 호출에 장애가 발생하면 fallback 메서드가 실행되도록 처리한다.

-   **구성 요소 및 동작 원리**
    -   **@CircuitBreaker 애노테이션**
        -   OrderService 클래스의 `createOrder` 메서드에 적용되어 user service 호출 시 장애가 감지되면 지정된 fallback 메서드 `fallbackGetUserGrade`가 호출
        -   fallback 메서드는 장애 상황에서도 주문 생성 로직을 진행하도록 처리하거나, 기본 할인 없이 주문을 완료하는 등의 대체 로직을 포함
    -   **실제 대체 동작**
        -   사용자 등급 정보를 가져오지 못할 경우, 장애 감지를 로그로 남기고 기본적인 주문 생성 프로세스를 실행하여 애플리케이션의 연속성을 유지