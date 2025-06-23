---
layout: post
title: "Service Instance 간의 통신"
date: 2025-06-21 22:00:00 +0900
categories: msa
background: '/img/posts/11.jpg'
---



-   MSA 구조에서의 인증 로직 구현
    
    # MSA 구조에서의 인증 로직 구현
    
    <aside> 💡
    
    MSA 구조에서는 개별 서비스들이 독립적으로 운영되며, 각 서비스가 자체적으로 인증 및 인가 로직을 처리할 수 있다. 그러나 외부 요청은 보통 API Gateway를 통해 들어오게 되며, 이를 활용한 중앙 집중식 인증 방식과 내부 서비스 간의 신뢰 체계 구성이 중요하다. 인증 로직을 구현할 때 고려해야 할 주요 사항은 JWT 토큰 검증 방식이다.
    
    </aside>
    
    ## 1. 재검증 방식
    
    각 모듈에서 클라이언트가 전달한 JWT 토큰 전체를 재검증하고 SecurityContext를 구성하는 방식이다. 이 방식은 다음과 같은 특징이 있다.
    
    -   **방어 심층(Defense in Depth) 전략:** 각 서비스가 독자적으로 JWT 토큰을 검증함으로써, 만약 API Gateway를 우회하여 직접 접근하는 요청이 발생할 경우에도 추가적인 보안 검증이 이루어진다. 이로써 서비스 간의 보안 취약점을 줄일 수 있다.
    -   **중복 코드 발생:** 서비스마다 동일한 JWT 토큰 검증 로직을 구현하게 되므로, 코드 중복이 발생할 수 있다. 이 경우 공통 라이브러리로 분리하여 관리할 수 있으나, 각 서비스가 독립적으로 해당 로직을 가져야 한다.
    -   **성능 부하:** 각 요청마다 모든 서비스에서 JWT 토큰 전체를 재검증할 경우, 불필요한 부하가 발생할 가능성이 있다. 특히 트래픽이 많은 서비스에서는 토큰 재검증이 성능에 영향을 줄 수 있다.
    -   **보안의 완전성:** JWT 토큰이 만료되었거나 변조된 경우 즉시 감지할 수 있으므로, 보안상의 완전성을 보장할 수 있다.
    
    즉, 완전 재검증 방식은 MSA 구조에서 “Defense in Depth”를 구현하는 데 유리하다. 다만, 각 서비스에 동일한 검증 로직이 중복되어 들어가는 단점과 이로 인한 시스템 부하를 고려해야 한다.
    
    ## 2. 신뢰 기반 방식: 헤더 정보 활용
    
    API Gateway에서 JWT 토큰의 기본 유효성 검증을 수행한 후, 토큰의 핵심 정보(예, 사용자 ID 및 사용자 역할)를 추출하여 HTTP 헤더(예, `X-User-Id`, `X-User-Role`)로 각 내부 서비스로 전달하는 방식이다. 이 방식의 특징은 다음과 같다.
    
    -   **코드 중복 감소:** API Gateway에서 한 번 검증된 정보를 각 서비스에 전달하므로, 개별 서비스에서 JWT 토큰 전체를 재검증할 필요가 없다. 이로써 코드 중복을 줄이고, 각 서비스의 구현이 간소화된다.
    -   **일관된 보안 모델 유지:** 모든 내부 서비스가 동일한 헤더 기반의 인증 정보를 활용하여 SecurityContext를 구성하므로, 보안 모델이 일관되게 유지된다. 각 서비스는 Gateway가 전달한 정보만을 신뢰하고 인가 로직을 실행하게 된다.
    -   **성능 향상:** 매번 모든 서비스에서 JWT 토큰 전체를 재검증하지 않아도 되므로, 시스템 부하가 감소하고 성능이 향상된다.
    -   **보안 취약점 고려:** API Gateway의 검증 로직에 문제가 발생하거나, Gateway 외부로부터의 접근이 허용되는 경우 보안 취약점이 발생할 수 있다. 따라서 Gateway의 보안이 가장 중요하며, 내부 서비스는 기본적으로 Pre-Authentication 방식(예, 헤더 정보를 기반으로 SecurityContext를 구성하는 필터)을 적용하여 최소한의 보안을 유지하는 것이 바람직하다.
    
    신뢰 기반 방식은 MSA 구조에서 성능 최적화와 코드 관리의 효율성을 높이는 데 유리하다. 단, 이 방식은 API Gateway가 모든 외부 요청에 대해 올바르게 JWT 검증을 수행한다는 전제 하에 작동하므로, Gateway의 보안 강화 및 모니터링에 신경을 써야 한다.
    
    ## 3. MSA 환경에서 인증 및 인가 구현 위치
    
    ### API Gateway
    
    -   **인증(Authentication) 집중 처리:**
        
        API Gateway는 모든 외부 요청의 진입점이므로, JWT 토큰의 유효성 검증과 사용자 정보(예, 사용자 ID, 역할)를 추출하는 전역 필터(GlobalFilter)를 통해 인증 처리를 수행한다.
        
        Gateway는 JWT 토큰이 유효한지를 확인하고, 검증된 정보를 HTTP 헤더에 추가하여 내부 서비스에 전달한다.
        
    -   **보안 로직의 중앙 집중화:**
        
        모든 외부 요청은 Gateway에서 먼저 처리되므로, 보안 정책을 중앙에서 일관되게 적용할 수 있다.
        
    
    ### 내부 서비스 (User, Order, 기타 서비스)
    
    -   **인가(Authorization) 처리 및 Pre-Authentication:**
        
        각 내부 서비스는 API Gateway가 전달한 인증 정보를 기반으로 SecurityContext를 구성한다.
        
        예를 들어, HeaderAuthenticationFilter와 같은 필터를 사용하여 `X-User-Id` 및 `X-User-Role` 헤더 값을 읽고, PreAuthenticatedAuthenticationToken을 생성하여 Spring Security Context에 설정한다.
        
    -   **추가 인가 정책 적용:**
        
        서비스별로 세부 리소스에 대한 접근 제어가 필요할 경우, 메소드 보안(@PreAuthorize, @PostAuthorize) 등을 통해 추가 인가 정책을 적용한다.
        
    
    ## 4. 정리
    
    MSA 구조에서 인증 로직을 구현하는 방법은 두 가지 방식인 완전 재검증 방식과 신뢰 기반 방식(헤더 정보 활용)으로 나뉜다.
    
    -   **완전 재검증 방식**은 각 서비스에서 JWT 토큰 전체를 재검증하여 보안의 완전성을 보장하지만, 코드 중복과 성능 부하가 발생할 수 있다.
    -   **신뢰 기반 방식**은 API Gateway에서 JWT 토큰을 검증한 후 핵심 정보를 헤더에 담아 각 내부 서비스에서 이를 활용하는 방법으로, 코드 중복을 줄이고 일관된 보안 모델을 유지할 수 있으며, 성능면에서도 유리하다.
    
    따라서 MSA 구조에서는 **인증(Authentication)은 API Gateway에서 집중 처리**하고, 내부 서비스에서는 Gateway가 전달한 인증 정보를 **신뢰하여 Pre-Authentication 방식으로 SecurityContext를 구성**하는 것이 바람직하다. 이를 통해 각 서비스는 Spring Security의 인가(Authorization) 기능을 효과적으로 활용할 수 있으며, 시스템 전체의 보안과 성능을 모두 만족시키는 구현이 가능하다.
    
-   User Service의 Spring Security 예제에서 변경 된 부분 체크
    
    # User Service의 Spring Security 에서 변경 된 부분 체크
    
    ### Eureka Client 설정
    
    -   build.gradle > dependencies > eureka client 추가
    -   application.yml > eureka 관련 설정 추가
    
    ### 인증 / 인가 설정
    
    -   JwtTokenProvider
        -   createToken, refreshToken : claim에 userId 추가
        -   AuthService의 createToken, refreshToken 호출부 수정
    -   SecurityConfig
        -   filterChain : JwtAuthenticationFilter 삭제 → HeaderAuthenticationFilter 추가
        -   api gateway에서 토큰 확인 후 userId와 role을 request header에 담아서 전송하도록 수정하므로 전달 받은 값 기준으로 SecurityContextHolder를 채워서 기능별 인증, 인가 확인이 일어나도록 함
    -   CumtomUserDetailsService → 삭제
    -   UserQueryController > getUserDetail
        -   @AuthenticationPrincipal String userId 로 변경
        -   UserQueryService > getUserDetail 매개변수 변경
        -   UserMapper > findUserById 추가, findUserByUsername 삭제
    
    ### 호출 URL 변경
    
    -   SecurityConfig
        -   filterChain : authorizeHttpRequest 설정에서 /api/v1 삭제
    -   XXXController
        -   @RequestMapping : /api/v1 삭제
    
    ### Order Service에서 호출할 추가 된 코드
    
    -   UserQueryController > getUserGrade
    -   UserQueryService > getUserGrade
    -   UserMapper(interface) > findUserById
    -   UserMapper.xml > findUserById

# 3. Service Instance 간의 통신

## 3-1. 스프링 Discovery Client로 서비스 인스턴스 검색

### 3-1-1. 스프링 Discovery Client란?

<aside> 💡

스프링 Discovery Client는 로드 밸런서(Spring Cloud Load Balancer)와 그 안에 등록된 서비스에 대해 가장 낮은 수준으로 접근할 수 있다. 즉, Discovery Client를 사용하면 스프링 클라우드 로드 밸런서 클라이언트에 등록된 모든 서비스와 해당 URL을 쿼리할 수 있다.

</aside>

### 3-1-2. 예제 코드

```java
@Component
public class OrderDiscoveryClient {

    @Autowired
    private DiscoveryClient discoveryClient; 

    public Order getOrder(String orderId) {
        RestTemplate restTemplate = new RestTemplate();
        List<ServiceInstance> instances = 
            discoveryClient.getInstances("order-service");

        if (instances.size() == 0) return null;
        String serviceUri = String.format(
            "%s/v1/order/%s", instances.get(0).getUri().toString(),
            orderId); 
    
        ResponseEntity<Order> restExchange = 
            restTemplate.exchange(
                serviceUri,
                HttpMethod.GET,
                null, Order.class, orderId);

        return restExchange.getBody();
    }
}


```

-   `DiscoveryClient` : 스프링 클라우드 로드 밸런서와 상호 작용한다. 유레카에 등록된 서비스의 모든 인스턴스를 검색하려면 getInstances() 메서드를 사용하고, ServiceInstance 객체 리스트를 얻어 오기 위해 찾으려는 서비스 키를 전달한다. ServiceInstance 클래스는 호스트 이름, 포트, URI 같은 서비스의 인스턴스 정보를 보관한다.
-   `RestTemplate`으로 Order 서비스를 호출하고 데이터를 조회할 수 있다.

### 3-1-3. RestTemplate 이란?

<aside> 💡

RestTemplate은 스프링 프레임워크에서 제공하는 HTTP 클라이언트 라이브러리로, RESTful 웹 서비스를 호출하는 데 사용되는 동기식 클라이언트이다. HTTP 메서드(GET, POST, PUT, DELETE 등)를 쉽게 사용할 수 있는 메서드를 제공하며 URI 템플릿 변수와 매개변수를 지원하여 동적 URL을 쉽게 구성할 수 있다. 요청/응답에 대한 객체 변환을 자동으로 처리하며 커스텀 헤더 관리, 에러 핸들링 등 다양한 기능을 제공한다.

</aside>

### 3-1-4. 단점

<aside> 💡

Discovery Client를 직접 호출하면 서비스 리스트를 얻게 되지만, 호출할 서비스 인스턴스를 선정할 책임은 사용자에게 있다. 즉, **스프링 클라우드 클라이언트 측 로드 밸런서를 이용하지 못한다.**

또한 코드에서 서비스를 호출하는 데 사용될 URL을 생성해야 한다. 즉, 서비스 메소드 내에서 **너무 많은 일을 한다.** 코드를 적게 작성하면 디버그할 코드가 줄어든다.

</aside>

## 3-2. 로드 밸런서를 지원하는 스프링 REST 템플릿으로 서비스 호출

### 3-2-1. 예제 코드

로드 밸런서를 지원하는 RestTemplate 클래스를 사용하려면 스프링 클라우드의 `@LoadBalanced` 애너테이션으로 RestTemplate 빈(bean)을 정의해야 한다.

```java
@SpringBootApplication
public class UserApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }

    @LoadBalanced 
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}

```

```java
@Component
public class OrganizationRestTemplateClient {
    @Autowired
    RestTemplate restTemplate;

    public Order getOrder(String orderId) {
        ResponseEntity<Order> restExchange =
            restTemplate.exchange(
                "<http://order-service/v1/>
                    order/{orderId}", 
                HttpMethod.GET, null, 
                Organization.class, orderId);

        return restExchange.getBody();
    }
}

```

-   스프링 클라우드 Discovery Client를 직접 사용하지 않는다.
-   restTemplate.exchange() 호출에 사용된 URL에서 서버 이름은 유레카에 Order 서비스를 등록할 때 사용된 Order 서비스 키의 애플리케이션 ID와 일치한다.

### 3-2-2. 예제 코드

<aside> 💡

로드 밸런서를 지원하는 RestTemplate 클래스는 전달된 URL을 파싱하고 서버 이름으로 전달된 것을 키로 사용하여 서비스의 인스턴스를 로드 밸런서에 쿼리한다. 실제 서비스 위치와 포트는 개발자에게 완전히 추상화된다. RestTemplate 클래스를 사용하면 스프링 클라우드 로드 밸런서는 서비스 인스턴스에 대한 모든 요청을 라운드 로빈 방식으로 부하 분산한다.

</aside>

## 3-3. 넷플릭스 Feign 클라이언트로 서비스 호출

### 3-2-1. Feign 클라이언트란?

<aside> 💡

Feign 라이브러리는 REST 서비스를 호출하는 데 RestTemplate 클래스와 다른 접근 방식을 취한다. 이 방식을 사용하려면 개발자는 먼저 자바 인터페이스를 정의한 후 스프링 클라우드 로드 밸런서가 호출할 유레카 기반 서비스를 매핑하기 위해 스프링 클라우드 애너테이션들을 추가해야 한다. 인터페이스를 정의하는 것 외에 서비스를 추가하려고 부가적으로 작성해야 할 코드는 없다.

</aside>

### 3-2-1. 예제 코드

```java
@FeignClient(name="order-service", url="localhost:8000") 
public interface OrganizationFeignClient {

    @RequestMapping(method=RequestMethod.GET, value="/v1/orders/{orderId}")
    Order getOrder(@PathVariable("organizationId") String orderId);
    
}

```

-   getOrder() 메서드를 정의하는 방법은 스프링 Controller 클래스에서 엔드포인트를 노출하는 방식과 똑같다.
    -   주문 서비스를 호출할 때 HTTP 동사와 엔드포인트를 노출하도록 매핑하는 @RequestMapping 애너테이션을 getOrder() 메서드에 정의한다.
    -   URL에 전달된 order ID를 @PathVariable 애너테이션을 사용하여 getOrder() 메서드의 orderId 매개변수에 매핑한다.
    -   주문 서비스의 호출 반환값은 getOrder() 메서드의 반환값으로 정의된 Order 클래스에 자동으로 매핑된다.

### 3-2-3. 에러 핸들링

<aside> 💡

표준 스프링 RestTemplate 클래스를 사용할 때 모든 서비스 호출에 대한 HTTP 상태 코드(status code)는 ResponseEntity 클래스의 getStatus() 메서드로 반환된다. 하지만 Feign 클라이언트를 사용하면 호출된 서비스에서 반환한 모든 HTTP 4xx – 5xx 상태 코드가 FeignException에 매핑된다. FeignException에는 특정 에러 메시지에 대해 파싱할 수 있는 JSON 내용이 포함되어 있다.

Feign은 사용자가 정의한 Exception 클래스에 에러를 재매핑하는 에러 디코더(decoder) 클래스를 작성할 수 있는 기능을 제공한다. 이 디코더를 작성하는 작업은 Feign 깃허브([**](https://github.com/Netflix/feign/wiki/Custom-error-handling)[https://github.com/Netflix/feign/wiki/Custom-error-handling**](https://github.com/Netflix/feign/wiki/Custom-error-handling**)) 등에서 예를 찾을 수 있다.

</aside>