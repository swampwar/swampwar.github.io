---
layout: post
title: SecurityContextHolder, SecurityContextPersistenceFilter
tags: [spring, security, Session, SecurityContextHolder, SecurityContextPersistenceFilter]
---

스프링 시큐리티를 공부하며 주요 필터에 대해 정리해본다.

## Session, SecurityContextHolder

인증이 완료되면 결과물인 `Authentication`이 `SecurityContextHolder`에 저장되며 필요시 가져올 수 있다. 컨트롤러, 서비스 등 어디에서나 인증 객체에 대해 아래 코드로 접근이 가능하다.

```java
// SecurityContextHolder의 SecurityContext에서 Authentication을 꺼낸다.
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
```

`SecurityContextHolder`는 인메모리 저장소이며 요청처리가 완료되면 사라진다. 
하지만 사라지기 전에 다른 저장소(디폴트 설정으로는 Session)에 `SecurityContext`를 저장하고, 다음 요청을 처리할 때는 이 저장소에서 정보를 꺼내 다시 `SecurityContextHolder` 를 메모리에 올려 처리하는 식이다.

`HttpSession`을 직접 사용하는 것이 아니라 추상화시킨 `SecurityContextHolder`를 사용함으로써 Redis, MongoDB 등으로 저장소를 변경할 수도 있다.(확장 가능한 포인트)

`SecurityContextHolder` 의 기본 `SecurityContext` 저장 전략은 `MODE_THREADLOCAL`로써, 해당 쓰레드에서만 접근이 가능하다. 만일 자식 쓰레드에서도 접근을 원할경우 `MODE_INHERITABLETHREADLOCAL`, 모든 쓰레드에서 접근을 원할경우 `MODE_GLOBAL`로 저장 전략을 변경한다.

```java
SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_THREADLOCAL); // 기본설정
SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_GLOBAL);
SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL);
```

참고 : [https://brunch.co.kr/@sbcoba/11](https://brunch.co.kr/@sbcoba/11)

## SecurityContextPersistenceFilter

![SecurityFilterChain]({{ "/assets/img/202007/SecurityFilterChain.png" | relative_url }})

위 필터체인의 최상단부에 위치하고 있는 필터이다. 요청 처리과정에서 `SecurityContext`을 생성하여 `SecurityContextHolder`에 저장하거나 저장소(디폴트는 세션)에 저장된 `SecurityContext`가 있다면 로드하여 `SecurityContextHolder`에 저장하는 역할을 한다.

저장소에서 로드시 `SecurityContextRepository` 인터페이스를 이용하는데 디폴트는 세션을 저장소로 하는 `HttpSessionSecurityContextRepository` 구현체를 이용한다. 만일 DB나 다른 저장소를 사용할 경우 저장소에 맞는 `SecurityContextRepository`를 구현하여 설정한다. (확장 포인트)

`SecurityContextHolder`의 기본 저장전략은 쓰레드 단위이므로 쓰레드끼리는 `SecurityContext`가 공유되지 않는다.

아래는 요청 유형에 따른 `SecurityContextPersistenceFilter`의 동작이다.

- 미인증 사용자의 자원 요청시
    1. 기존에 저장된 `SecurityContext`가 없으므로 새로운 `SecurityContext`를 생성하여 `SecurityContextHolder`에 저장
    2. 이후 `AnonymousAuthenticationFilter` 가 `AnonymousAuthenticationToken`을 `SecurityContext`에 저장
    
- 인증(로그인) 요청시
    1. 기존에 저장된 `SecurityContext`가 없으므로 새로운 `SecurityContext`를 생성하여 `SecurityContextHolder`에 저장
    2. `UsernamePasswordAuthenticationFilter`가 인증에 성공하면 `UsernamePasswordAuthenticationToken` 을 `SecurityContext` 에 저장
    3. `SecurityContextHolder`의 `SecurityContext`를 저장소(세션)에 저장(다음 요청에서 활용하기 위해)

- 인증된 사용자의 자원 요청시
    1. 저장소(세션)에서 기존에 저장된 `SecurityContext`를 로드하여 `SecurityContextHolder`에 저장

모든 요청의 마무리에는 `SecurityContextPersistenceFilter`가 `SecurityContextHolder`의 `SecurityContext`를 정리한다. 위 이미지의 나가는 방향 화살표에서 최종처리가 `SecurityContextPersistenceFilter`를 거치는 것을 볼 수 있다.

```java
SecurityContextHolder.clearContext();
```

