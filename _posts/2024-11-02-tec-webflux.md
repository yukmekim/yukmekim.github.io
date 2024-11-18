---
title: 'MVC와 WebFlux의 차이, 고속 주행을하는 WebFlux..?'
description: 웹 애플리케이션을 만들다 보면, 어떤 프레임워크를 사용할지 고민하게 된다. 특히 Spring Framework의 두 가지 대표적인 옵션인 Spring MVC와 WebFlux는 서로 다른 접근 방식을 가지고 있는데, 오늘은 이 두 프레임워크의 차이점을 살펴보면서, WebFlux가 왜 "고속 주행"을 할 수 있는지 알아보려고 한다.
categories:
 - Java
tags:
 - Java
---
**MVC**와 **WebFlux**를 비교하기 전 두 옵션이 가진 특징을 이해하기 위해 **동기**와 **비동기**에 대해 간단하게 정리를 하고 넘어가겠다.

## 동기와 비동기의 차이

**동기(Synchronous)**: 요청을 보낸 후, 그 요청의 처리가 완료될 때까지 기다리는 방식이다. 즉, 하나의 작업이 끝나야 다음 작업을 시작할 수 있다. 이 방식은 구현이 간단하고 예측하기 쉬운 장점이 있지만, 대기 시간이 길어질 수 있다는게 단점이 될수있다.

**비동기(Asynchronous)**: 동기와 반대되는 개념으로 요청을 보낸 후, 그 처리가 완료될 때까지 기다리지 않고 다른 작업을 수행할 수 있는 방식이다. 즉, 요청을 보낸 후 응답이 오기전에 다음 작업을 진행할 수 있다는 뜻이다. 이 방법은 서버 자원을 효율적으로 사용할 수 있어 높은 성능을 제공하지만, 코드의 흐름이 복잡해질 수 있는 단점이 있다.

두 가지 방식이 가지는 장단점이 있으므로, 높은 성능과 빠른 처리를 한다고 무조건적으로 비동기 처리가 옳지는 않다. 이 부분을 주의하며 본문으로 넘어가 MVC와 WebFlux의 차이를 살펴보자.


## MVC와 WebFlux의 차이
### MVC: 느긋한 드라이브
MVC는 전통적인 블로킹 방식으로 동작하는 웹 프레임워크다. 클라이언트의 요청이 들어오면, **해당 요청을 처리할 스레드가 할당**되고, 이 스레드는 요청이 끝날 때까지 기다리게 된다. **안정적**이고 구현이 간단하지만, 요청이 많아지면 성능 저하가 발생할 수 있다. 특히 여러 요청이 동시에 들어오면, 스레드가 차단되어 리소스 낭비 되며, **스레드 풀이 가득 차서 새로운 요청을 받지 못하는 상황(Thread Pool Hell)**이 발생할 수도 있다. 그래서 Spring MVC는 CRUD 애플리케이션 같은 안정적이고 보편적인 웹 서비스에 잘 맞는다고 한다.

### WebFlux: 고속 주행
반대로 WebFlux는 비동기적이고 논블로킹 방식으로 작동하는 웹 프레임워크다. 요청이 들어오면, **스레드가 블로킹되지 않고 다른 작업을 수행**할 수 있다. 이를 위해 Mono와 Flux와 같은 리액티브 스트림을 사용하여 비동기적 처리를 제공한다. 이러한 특징 덕분에 많은 요청을 **효율적**으로 처리할 수 있고, **서버의 리소스를 최대한 활용**할 수 있다. 즉, WebFlux는 **"고속 주행"**처럼 빠르게 데이터를 처리할 수 있는 장점이 있다.

### 간단한 예제를 통한 성능 테스트

**MVC**
```java
@RestController
@RequestMapping("/api/v2/mvcApi")
public class MvcAPI {
    private final MemberService memberService;

    public MvcAPI(MemberService memberService) {
        this.memberService = memberService;
    }

    @GetMapping("/helloMvc")
    public Map<String, String> helloMvc() {
        return Map.of("status", "200");
    }

    @GetMapping("/mvcData")
    public ResponseEntity<Response> testFlux(final String id) {
        HttpHeaders header = new HttpHeaders();
        header.setContentType(new MediaType(MediaType.APPLICATION_JSON, StandardCharsets.UTF_8));

        Optional<Member> result = Optional.of(memberService.getUserById(id).orElse(new Member()));

        Response response = Response.builder()
                .status(StatusEnum.OK)
                .message("success")
                .data(result)
                .build();

        return new ResponseEntity<>(response, header, HttpStatus.OK);
    }
}
```

**WebFlux**
```java
@RestController
@RequestMapping("/api/v2/webFluxApi")
public class WebFluxAPI {
    private final MemberService memberService;

    public WebFluxAPI(MemberService memberService) {
        this.memberService = memberService;
    }

    @GetMapping("/helloWebFlux")
    public Mono<Map<String, String>> helloWebFlux() {
        return Mono.just(Map.of("status", "OK"));
    }

    @GetMapping("/webFluxData")
    public Mono<Response> webFluxData(final String id) {
        Optional<Member> result = Optional.of(memberService.getUserById(id).orElse(new Member()));

        Response response = Response.builder()
                .status(StatusEnum.OK)
                .message("success")
                .data(result)
                .build();

        return Mono.just(response);
    }
}
```

application.yml
```yml
server:
  port: 8099
  tomcat:
    max-connections: 10000
    accept-count: 1000
    threads:
      max: 3000
      min-spare: 1000

spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://dev.portdemo.com:3307/dev_port
    username: port_user
    password: XXXX
  jpa:
    generate-ddl: on
    hibernate:
      ddl-auto: create

logging:
  level:
    root: INFO
```

스레드 최소 설정 값은 1000으로 **apache jmeter**를 사용해서 동시 요청 테스트를 진행했다.

![Desktop Preview](/assets/images/post/webflux/jmeter_test_case_1.png)

**MVC**
![Desktop Preview](/assets/images/post/webflux/jmeter_test_case_2.png)

throughput(TPS) : 10352.0/sec  
평균 응답시간 : 167 ~ 200ms

**WebFlux**
![Desktop Preview](/assets/images/post/webflux/jmeter_test_case_3.png)

throughput(TPS) : 12372.2/sec  
평균 응답시간 : 89 ~ 137ms

간단한 예제 코드와 jmeter를 사용해서 성능 비교를 해봤는데

Summary Report를 보면 WebFlux가 MVC보다 응답시간이 빠르면서 초당 처리한 트랜잭션양이 많은걸 알 수 있다.
가벼운 조회 쿼리를 사용하였는데도 성능차이가 눈에 보일 정도이다.

다음에 기회가 된다면 redis 캐시를 사용해서 테스트 결과를 도출해보도록 하겠다.

## 결론
**MVC**와 **WebFlux**는 각기 다른 특징과 장점이 있다.

### 차이점 정리

- 처리 방식:
    - **Spring MVC**: 요청을 처리할 때 스레드가 블로킹됨.
    - **WebFlux**: 요청을 비동기적으로 처리하여 스레드가 자유롭게 사용됨.
- 성능:
    - **Spring MVC**: 트래픽이 많아지면 성능 저하가 발생할 수 있음.
    - **WebFlux**: 높은 트래픽을 효율적으로 처리, **"고속 주행"** 가능.
- 사용 사례:
    - **Spring MVC**: 안정적인 CRUD 애플리케이션에 적합.
    - **WebFlux**: 실시간 데이터 처리와 대규모 트래픽을 요구하는 애플리케이션에 적합.

비유적으로 말하자면, **Spring MVC**는 안정적이고 예측할 수 있는 경로를 제공하여 간단한 CRUD 애플리케이션이나 안정적인 웹 서비스를 구축할 때 유리하지만, 트래픽이 많아지면 성능 저하가 발생할 수 있기 때문에 **"느긋한 드라이브"**라 할 수 있는 반면에, **WebFlux**는 비동기 방식으로 설계되어 있어 요청을 빠르게 처리할 수 있으며, 대규모 트래픽을 효과적으로 관리할 수 있고 실시간 데이터 처리와 같은 복잡한 애플리케이션에서 더 적합하다는 데 있어 **"고속 주행"**이라고 비유해 봤다.

결국, 어떤 프레임워크를 선택할지는 프로젝트의 요구 사항과 환경에 따라 달라져야 하므로, 무조건 하나를 선택할 수는 없다.