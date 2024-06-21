## 프록시
#### 프록시의 주요 기능
프록시를 통해서 할 수 있는 일은 크게 2가지로 구분할 수 있다.
* 접근 제어
    * 권한에 따른 접근 차단
    * 캐싱
    * 지연 로딩
* 부가 기능 추가
    * 원래 서버가 제공하는 기능에 더해서 부가 기능을 수행한다.
    * 예) 요청 값이나, 응답 값을 중간에 변형한다.
    * 예) 실행 시간을 측정해서 추가 로그를 남긴다.

#### GOF 디자인 패턴
둘다 프록시를 사용하는 방법이지만 GOF 디자인 패턴에서는 이 둘을 의도(intent)에 따라서 프록시 패턴과 데코레이터 패턴으로 구분한다.
* 프록시 패턴: 접근 제어가 목적
* 데코레이터 패턴: 새로운 기능 추가가 목적

#### 의도(intent)
사실 프록시 패턴과 데코레이터 패턴은 그 모양이 거의 같고, 상황에 따라 정말 똑같을 때도 있다. 그러면 둘을 어떻게 구분하는 것일까?  
디자인 패턴에서 중요한 것은 해당 패턴의 겉모양이 아니라 그 패턴을 만든 의도가 더 중요하다. 따라서 의도에 따라 패턴을 구분한다.  
* 프록시 패턴의 의도: 다른 개체에 대한 접근을 제어하기 위해 대리자를 제공  
* 데코레이터 패턴의 의도: 객체에 추가 책임(기능)을 동적으로 추가하고, 기능 확장을 위한 유연한 대안 제공  

##### 정리
프록시를 사용하고 해당 프록시가 접근 제어가 목적이라면 프록시 패턴이고, 새로운 기능을 추가하는 것이 목적이라면 데코레이터 패턴이 된다.  

#### JDK 동적 프록시
지금까지 프록시를 적용하기 위해 적용 대상의 숫자 만큼 많은 프록시 클래스를 만들었다. 적용 대상이 100개면 프록시 클래스도 100개 만들었다. 
그런데 앞서 살펴본 것과 같이 프록시 클래스의 기본 코드와 흐름은 거의 같고, 프록시를 어떤 대상에 적용하는가 정도만 차이가 있었다. 
쉽게 이야기해서 프록시의 로직은 같은데, 적용 대상만 차이가 있는 것이다.  

이 문제를 해결하는 것이 바로 동적 프록시 기술이다.
동적 프록시 기술을 사용하면 개발자가 직접 프록시 클래스를 만들지 않아도 된다. 이름 그대로 프록시 객체를 동적으로 런타임에 개발자 대신 만들어준다. 
그리고 동적 프록시에 원하는 실행 로직을 지정할 수 있다.

##### JDK 동적 프록시 InvocationHandler
JDK 동적 프록시에 적용할 로직은 InvocationHandler 인터페이스를 구현해서 작성하면 된다.  
JDK 동적 프록시가 제공하는 InvocationHandler 
```java
package java.lang.reflect;
public interface InvocationHandler {
public Object invoke(Object proxy, Method method, Object[] args)
throws Throwable;
}
```

제공되는 파라미터는 다음과 같다.
* Object proxy : 프록시 자신
* Method method : 호출한 메서드
* Object[] args : 메서드를 호출할 때 전달한 인수


#### CGLIB
##### CGLIB: Code Generator Library
* CGLIB는 바이트코드를 조작해서 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리이다.
* CGLIB를 사용하면 인터페이스가 없어도 구체 클래스만 가지고 동적 프록시를 만들어낼 수 있다.
* CGLIB는 원래는 외부 라이브러리인데, 스프링 프레임워크가 스프링 내부 소스 코드에 포함했다. 따라서 스프링을 사용한다면 별도의 외부 라이브러리를 추가하지 않아도 사용할 수 있다.

##### MethodInterceptor -CGLIB 제공 
```java
package org.springframework.cglib.proxy;
public interface MethodInterceptor extends Callback {
Object intercept(Object obj, Method method, Object[] args, MethodProxy
proxy) throws Throwable;
}
```
* obj : CGLIB가 적용된 객체
* method : 호출된 메서드
* args : 메서드를 호출하면서 전달된 인수
* proxy : 메서드 호출에 사용

