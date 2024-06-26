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

#### 프록시 팩토리
인터페이스가 있는 경우에는 JDK 동적 프록시를 적용하고, 그렇지 않은 경우에는 CGLIB를 적용

##### Advice 만들기
Advice 는 프록시에 적용하는 부가 기능 로직이다.  
이것은 JDK 동적 프록시가 제공하는 InvocationHandler 와 CGLIB가 제공하는 MethodInterceptor 의 개념과 유사하다.
둘을 개념적으로 추상화 한 것이다. 프록시 팩토리를 사용하면 둘 대신에 Advice 를 사용하면 된다.  
Advice 를 만드는 방법은 여러가지가 있지만, 기본적인 방법은 다음 인터페이스를 구현하면 된다.  
MethodInterceptor - 스프링이 제공하는 코드   
```java
package org.aopalliance.intercept;
public interface MethodInterceptor extends Interceptor {
Object invoke(MethodInvocation invocation) throws Throwable;
}
```
* MethodInvocation invocation  
    * 내부에는 다음 메서드를 호출하는 방법, 현재 프록시 객체 인스턴스, args , 메서드 정보 등이 포함되어 있다.
      기존에 파라미터로 제공되는 부분들이 이 안으로 모두 들어갔다고 생각하면 된다.
* CGLIB의 MethodInterceptor 와 이름이 같으므로 패키지 이름에 주의하자
    * 참고로 여기서 사용하는 org.aopalliance.intercept 패키지는 스프링 AOP 모듈( spring-aop )안에 들어있다.  
* MethodInterceptor 는 Interceptor 를 상속하고 Interceptor 는 Advice 인터페이스를 상속한다.

##### 프록시 팩토리를 통한 프록시 적용 확인
프록시 팩토리로 프록시가 잘 적용되었는지 확인하려면 다음 기능을 사용하면 된다.
* AopUtils.isAopProxy(proxy) : 프록시 팩토리를 통해서 프록시가 생성되면 JDK 동적 프록시나, CGLIB 모두 참이다.
* AopUtils.isJdkDynamicProxy(proxy) : 프록시 팩토리를 통해서 프록시가 생성되고, JDK 동적 프록시 인 경우 참
* AopUtils.isCglibProxy(proxy) : 프록시 팩토리를 통해서 프록시가 생성되고, CGLIB 동적 프록시인 경우 참

#### 포인트컷, 어드바이스, 어드바이저
스프링 AOP를 공부했다면 다음과 같은 단어를 들어보았을 것이다. 항상 잘 정리가 안되는 단어들인데, 단순하지만 중
요하니 이번에 확실히 정리해보자.
* 포인트컷( Pointcut ): 어디에 부가 기능을 적용할지, 어디에 부가 기능을 적용하지 않을지 판단하는 필터링 로직이다.
  주로 클래스와 메서드 이름으로 필터링 한다. 이름 그대로 어떤 포인트(Point)에 기능을 적용할지 하지 않을지 잘라서(cut) 구분하는 것이다.
* 어드바이스( Advice ): 이전에 본 것 처럼 프록시가 호출하는 부가 기능이다. 단순하게 프록시 로직이라 생각하면 된다.
* 어드바이저( Advisor ): 단순하게 하나의 포인트컷과 하나의 어드바이스를 가지고 있는 것이다. 쉽게 이야기해서 포인트컷1 + 어드바이스1이다.  

정리하면 부가 기능 로직을 적용해야 하는데, 포인트컷으로 어디에? 적용할지 선택하고, 어드바이스로 어떤 로직을 적용할지 선택하는 것이다.
그리고 어디에? 어떤 로직?을 모두 알고 있는 것이 어드바이저이다.

##### Pointcut 관련 인터페이스 - 스프링 제공
```java
public interface Pointcut {
  ClassFilter getClassFilter();
  MethodMatcher getMethodMatcher();
}
public interface ClassFilter {
  boolean matches(Class<?> clazz);
}
public interface MethodMatcher {
   boolean matches(Method method, Class<?> targetClass);
//..
}
```

##### 스프링이 제공하는 포인트컷
스프링은 무수히 많은 포인트컷을 제공한다.
* NameMatchMethodPointcut : 메서드 이름을 기반으로 매칭한다. 내부에서는 PatternMatchUtils 를 사용한다.
    * 예) `*`xxx`*` 허용
* JdkRegexpMethodPointcut : JDK 정규 표현식을 기반으로 포인트컷을 매칭한다.
* TruePointcut : 항상 참을 반환한다.
* AnnotationMatchingPointcut : 애노테이션으로 매칭한다.
* AspectJExpressionPointcut : aspectJ 표현식으로 매칭한다

#### 빈 후처리기
빈 후처리기는 빈을 조작하고 변경할 수 있는 후킹 포인트이다.  
이것은 빈 객체를 조작하거나 심지어 다른 객체로 바꾸어 버릴 수 있을 정도로 막강하다.  
여기서 조작이라는 것은 해당 객체의 특정 메서드를 호출하는 것을 뜻한다.  
일반적으로 스프링 컨테이너가 등록하는, 특히 컴포넌트 스캔의 대상이 되는 빈들은 중간에 조작할 방법이 없는데, 빈  
후처리기를 사용하면 개발자가 등록하는 모든 빈을 중간에 조작할 수 있다. 이 말은 빈 객체를 프록시로 교체하는 것도  
가능하다는 뜻이다.  

##### BeanPostProcessor 인터페이스 - 스프링 제공 
```java
public interface BeanPostProcessor {
Object postProcessBeforeInitialization(Object bean, String beanName) throws
BeansException
Object postProcessAfterInitialization(Object bean, String beanName) throws
BeansException
}
```
* 빈 후처리기를 사용하려면 BeanPostProcessor 인터페이스를 구현하고, 스프링 빈으로 등록하면 된다.
* postProcessBeforeInitialization : 객체 생성 이후에 @PostConstruct 같은 초기화가 발생하기전에 호출되는 포스트 프로세서이다.  
* postProcessAfterInitialization : 객체 생성 이후에 @PostConstruct 같은 초기화가 발생한 다음에 호출되는 포스트 프로세서이다.

##### AToBPostProcessor
* 빈 후처리기이다. 인터페이스인 BeanPostProcessor 를 구현하고, 스프링 빈으로 등록하면 스프링 컨테이너가 빈 후처리기로 인식하고 동작한다.
* 이 빈 후처리기는 A 객체를 새로운 B 객체로 바꿔치기 한다. 파라미터로 넘어오는 빈( bean ) 객체가 A 의 인스턴
  스이면 새로운 B 객체를 생성해서 반환한다. 여기서 A 대신에 반환된 값인 B 가 스프링 컨테이너에 등록된다. 
  다음 실행결과를 보면 beanName=beanA , bean=A 객체의 인스턴스가 빈 후처리기에 넘어온 것을 확인할 수 있다.

#### 스프링이 제공하는 빈 후처리기
자동 프록시 생성기 - AutoProxyCreator
* 앞서 이야기한 스프링 부트 자동 설정으로 AnnotationAwareAspectJAutoProxyCreator 라는 빈 후처리 기가 스프링 빈에 자동으로 등록된다.
* 이름 그대로 자동으로 프록시를 생성해주는 빈 후처리기이다.
* 이 빈 후처리기는 스프링 빈으로 등록된 Advisor 들을 자동으로 찾아서 프록시가 필요한 곳에 자동으로 프록시를 적용해준다.
* Advisor 안에는 Pointcut 과 Advice 가 이미 모두 포함되어 있다. 따라서 Advisor 만 알고 있으면 그 안  
  에 있는 Pointcut 으로 어떤 스프링 빈에 프록시를 적용해야 할지 알 수 있다. 그리고 Advice 로 부가 기능을  
  적용하면 된다.

> 참고
> AnnotationAwareAspectJAutoProxyCreator 는 @AspectJ와 관련된 AOP 기능도 자동으로 찾아서
> 처리해준다.
> Advisor 는 물론이고, @Aspect 도 자동으로 인식해서 프록시를 만들고 AOP를 적용해준다.

![자동프록시생성기](./src/main/resources/img/자동프록시생성기.png)
자동 프록시 생성기의 작동 과정을 알아보자
1. 생성: 스프링이 스프링 빈 대상이 되는 객체를 생성한다. ( @Bean , 컴포넌트 스캔 모두 포함)
2. 전달: 생성된 객체를 빈 저장소에 등록하기 직전에 빈 후처리기에 전달한다.
3. 모든 Advisor 빈 조회: 자동 프록시 생성기 - 빈 후처리기는 스프링 컨테이너에서 모든 Advisor 를 조회한다.
4. 프록시 적용 대상 체크: 앞서 조회한 Advisor 에 포함되어 있는 포인트컷을 사용해서 해당 객체가 프록시를
   적용할 대상인지 아닌지 판단한다. 이때 객체의 클래스 정보는 물론이고, 해당 객체의 모든 메서드를 포인트컷에
   하나하나 모두 매칭해본다. 그래서 조건이 하나라도 만족하면 프록시 적용 대상이 된다. 예를 들어서 10개의 메서
   드 중에 하나만 포인트컷 조건에 만족해도 프록시 적용 대상이 된다.
5. 프록시 생성: 프록시 적용 대상이면 프록시를 생성하고 반환해서 프록시를 스프링 빈으로 등록한다. 만약 프록
   시 적용 대상이 아니라면 원본 객체를 반환해서 원본 객체를 스프링 빈으로 등록한다.
6. 빈 등록: 반환된 객체는 스프링 빈으로 등록된다.

##### 포인트컷은 2가지에 사용된다.
1. 프록시 적용 여부 판단 - 생성 단계
   자동 프록시 생성기는 포인트컷을 사용해서 해당 빈이 프록시를 생성할 필요가 있는지 없는지 체크한다.
   클래스 + 메서드 조건을 모두 비교한다. 이때 모든 메서드를 체크하는데, 포인트컷 조건에 하나하나 매칭해
   본다. 만약 조건에 맞는 것이 하나라도 있으면 프록시를 생성한다.
   예) orderControllerV1 은 request() , noLog() 가 있다. 여기에서 request() 가 조건에
   만족하므로 프록시를 생성한다.
   만약 조건에 맞는 것이 하나도 없으면 프록시를 생성할 필요가 없으므로 프록시를 생성하지 않는다.
2. 어드바이스 적용 여부 판단 - 사용 단계
   프록시가 호출되었을 때 부가 기능인 어드바이스를 적용할지 말지 포인트컷을 보고 판단한다.
   앞서 설명한 예에서 orderControllerV1 은 이미 프록시가 걸려있다.
   orderControllerV1 의 request() 는 현재 포인트컷 조건에 만족하므로 프록시는 어드바이스를 먼
   저 호출하고, target 을 호출한다.
   orderControllerV1 의 noLog() 는 현재 포인트컷 조건에 만족하지 않으므로 어드바이스를 호출하지
   않고 바로 target 만 호출한다.

#### AspectJExpressionPointcut
AspectJ라는 AOP에 특화된 포인트컷 표현식을 적용할 수 있다. AspectJ 포인트컷 표현식과 AOP는 조금 뒤에 자세
히 설명하겠다. 지금은 특별한 표현식으로 복잡한 포인트컷을 만들 수 있구나 라고 대략 이해하면 된다.

