---
layout: post
author: "
jeoun"
---



# Spring Cloud 끄적여보기

## spring cloud?

간단하게 분산 시스템을 구축하기 위한 스프링 기반의 프레임워크라고 생각하면 된다.
Spring Cloud는 간단하고 빠르게 분산시스템 패턴으로 시스템들을 구축할수 있게 제공하고 있다.

기본적으로 spring boot기반으로 돌아가고 있기때문에
개발자는 빠르게 개발하고 적용할수 있다는 장점도 가지고 있다.
(많은 레퍼런스도 있고, Spring Boot가 갖는 여러가지 장점으로 인해.)

http://projects.spring.io/spring-cloud/ 
https://www.jhipster.tech/microservices-architecture/

실제 프로젝트에 적용할때는 위 2가지 문서를 주로 참고해서 진행하였다.

## 어떤 구성?

Spring Cloud는 여러가지 이루어져서 여러가지 기능들을 제공하고 있다
간단히 나열해보면
Service Discovery, Configuration Server, Api Gateway, Oauth2, Load Balancing... 등등 
훨씬 다양한 구성요소로 이루어져있다

##### Service Discovery

클라이언트 사이드에서 로드발란싱과 라우팅을 좀더 잘 간단하게 할수 있도록 디렉토리화 하고 있다
여기서 클라이언트는 외부 클라이언트가 아니라 eureka client들을 의미한다
eureka client들은 기동하는 시점에 service discovery에 어떤 프로젝트로 실행되고 있는지 등록을 하게된다.
그리고 주기적으로 service discovery에게 다른 eureka client들의 정보를 받아간다.

이런 정보를 토대로 각기 다른 종류의 eureka client에게 ribbon과 같은 기술을 사용해서 요청을 간단히 보낼수도 있고,
서버가 죽을 경우 자동으로 request를 받지 않도록 빠질수도 있다.

Service Discovery의 경우 더 많은 기능들을 제공하기 때문에 문서들을 보면서 하나씩 따라해보면 좋다.

##### Circuit Breaker

microservice들을 모니터링하며 에러가 발생되는 특정 임계치를 넘어가게되면 자동적으로 접속을 차단하는 기능을 제공한다
특정 시스템의 오류가 다른 시스템들로의 시스템 이상으로 전파를 막기위해서 사용되는 기능이다

##### Configuration Server

프로젝트 별로 config들을 가지고 있지 않고 한군데서 중앙 관리형태로 config를 관리하는 거야.
여러 microservice에서 각각 config를 수정하지 않고 여기서만 수정하게 되면 모든 microservice에 자동적으로 적용하게 되지. 

##### Api gateway

각각의 microservice들은 외부에서 직접적으로 호출을 받지 않아.. 
대신 api gateway를 두고 외부와 통신하는 구성을 별도로 두고 있어. 
쉽게 생각하면 외부 client(app, browser)들을 위한 api end point야.

##### Oauth2

인증을 위한 프로젝트이야.



이외에도 엄청 다양한 프로젝트들로 구성되어있어..
아직 spring cloud의 기능들을 모두 써보진 못했지만.. 지금 개발중인 시스템에 하나씩 적용을 해보고 있는 단계이야.
아직은 msa구조를 할만큼 크거나 많은 트래픽이 몰리는 시스템도 아니지만.. 언젠간 커지겠지?
그래도 기존 구조가 기능별로 프로젝트가 쪼개져서 각기 다른 서버에서 실행되고 있었기에 spring cloud로 묶어서 보니.
관리하기에는 좀더 편해진 부분도 있는것같어.
