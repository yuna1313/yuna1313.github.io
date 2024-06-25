---
published: true
title: "[Spring] Filter와 Interceptor 차이"
categories:
  - Spring
tags:
  - filter
  - interceptor
  - Spring

toc: true
toc_sticky: true

date: 2023-11-05
last_modified_at: 2023-11-05
---

새로운 프로젝트를 시작하면서 Redis를 통하여 세션 관리를 하도록 설정하였다. 그래서 세션이 없어도 사용할 수 있는 API 이외에는 세션 체크를 공통으로 처리해줘야 했다. <br>
처음에 잘 모르고 Filter로 무작정 설정하였다가 복잡한 세팅으로 애를 먹고 Interceptor로 수정하였는데 그 과정에서 알게 된 Filter와 Interceptor의 차이점을 작상하려 한다.

---

## 1.Filter란?
Filter는 Dispatcher Servelt에 요청이 전달되기 전/후에 URL 패턴에 맞는 모든 요청에 대하여 부가 작업 처리할 수 있는 기능을 제공한다. (Dispatcher Servelt에 대한 내용은 추후에 다시 다루도록 하겠다!)
![Filter](https://github.com/yuna1313/yuna1313.github.io/assets/93983333/b8510a9c-27e6-4a9d-a389-8373025867c0)

### Filter의 메소드
아래는 Filter의 Interface이며 init(), doFilter(), destroy() 총 3가지의 메소드로 구성되어있다.
```
public interface Filter {

    public default void init(FilterConfig filterConfig) throws ServletException {}
    
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;
            
    public default void destroy() {}

}
```
1. init() : Filter 객체를 초기화하고 서비스에 추가하기 위해 사용하기 위한 메소드이다.
2. doFilter() : URL 패턴에 맞는 모든 HTTP 요청이 Dispatcher Servelt으로 전달되지 전에 웹 컨테이너에 의해 실행되는 메소드이다. <span style='background-color: #fff5b1'>FilterChain의 doFilter를 통해 다음 대상으로 요청을 전달하게 되는데 chain.doFilter 하기 전에 필요한 처리 과정을 진행하면 된다.</span>
3. destroy() : Filter 객체를 서비스에서 제거하고 사용하는 자원을 반환하기 위해 사용하는 메소드이다.

## 2.Interceptor란?
intercept라는 단어가 '가로채기'라는 뜻이다. 그래서 Interceptor는 Dispatcher Servelt이 Controller를 호출하기 전후로 가로채서 요청과 응답을 참조하거나 가공할 수 있는 기능을 제공한다.<br>
추가로 만약 Interceptor가 여러 개 등록되었으면 순차적으로 Interceptor를 수행 후 Controller를 실행하게 된다.
![interceptor](https://github.com/yuna1313/yuna1313.github.io/assets/93983333/42f8b801-d647-4737-9b74-6e4531a87198)

### Interceptor의 메소드
아래는 Interceptor의 Interface이며 preHandle (), postHandle(), afterCompletion() 총 3가지의 메소드로 구성되어있다.
```
public interface HandlerInterceptor {

    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws Exception {
        
        return true;
    }

    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
        @Nullable ModelAndView modelAndView) throws Exception {
    }

    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
        @Nullable Exception ex) throws Exception {
    }
}
```
1. preHandle() : Controller가 호출되기 전에 실행되는 메소드이다. 그래서 Controller 이전에 처리해야 하는 전처리나 가공을 해당 메소드에서 실행할 수 있다. <span style='background-color: #fff5b1'>해당 메소드는 return 타입이 boolean인데 true일 경우 다음 단계로 진행되지만 false인 경우 다음 단계(다음 Interceptor, Controller 등의 작업)를 진행하지 않는다. </span>
2. postHandle() : Controller가 호출된 이후에 실행되는 메소드이다. 그래서 Controller 이후에 후처리가 필요할 경우 해당 메소드에서 실행할 수 있다.
3. afterCompletion() : 모든 View에서 최종 결과를 생성하는 일을 포함해 모든 작업이 완료된 후에 실행된다. 요청 처리 중에 사용한 리소스를 반환할 때 사용할 수 있다.

## 3.Filter와 Interceptor의 사용 용도
가장 큰 차이점은 <span style='color: red'>Request/Response 객체 조작 가능 여부</span>와 <span style='color: red'>사용 용도</span>로 확인할 수 있다.

### 3-1.Request/Response 객체 조작 가능 여부
Filter의 경우 객체를 조작할 수 있다. 다음 필터를 호출하기 위해 필터 페이닝(다음 필터 호출)을 해주서야 하는데 이 때 원하는 Request/Response 객체를 넣어줄 수 있다.
```
public MyFilter implements Filter {

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        chain.doFilter(request, response);   
    }
    
}
```
Interceptor의 경우 객체를 조작할 수 없다. Dispatcher Servelt이 여러 Interceptor 목록을 가지고 있고 순차적으로 실행 시킨다. 이 때 true를 반환해야 다음 Interceptor나 Controller가 실행되고 false일 경우 중단되므로 다른 Request/Response 객체를 넣어줄 수 없다.
```
public class MyInterceptor implements HandlerInterceptor {

    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        return true;
    }
}
```

### 3-2.사용 용도
<span style='background-color: #fff5b1'>Filter의 경우 기본적으로 스프링과 무관하게 전역적으로 처리해야 하는 작업들을 처리할 수 있다.</span>
Filter는 Interceptor보다 앞단에서 동작하므로 보안 검사(XSS Filter)를 하여 올바른 요청이 아닐 경우 차단할 수 있다.
1. 공통된 보안 및 인증/인가 관련 작업
2. 모든 요청에 대한 로깅 또는 감사
3. 이미지/데이터 압축 및 문자열 인코딩
4. Spring과 분리되어야 하는 기능

<span style='background-color: #fff5b1'>Interceptor의 경우 클라이언트의 요청과 관련되어 전역적으로 처리해야 하는 작업들을 처리할 수 있다.</span> 대표적으로 세부적으로 적용해야 하는 인증이나 인가와 같이 클라이언트 요청과 관련된 작업 등이 있다.
1. 세부적인 보안 및 인증/인가 공통 작업
2. API 호출에 대한 로깅 또는 감사
3. Controller로 넘겨주는 정보(데이터)의 가공

## 4.정리 이미지
![summary](https://github.com/yuna1313/yuna1313.github.io/assets/93983333/cd71c929-1b10-40ac-9264-278dbab76f11)

---

위 내용을 알아보면서 내가 세션 처리하려고 작성했던 내용은 Interceptor에서 처리하는 것이 맞다는 것을 알게되었다. '2년 차를 접어들고 있는데 아직도 모르는 것이 투성이라니..' 라는 생각과 '지금이라도 알았으면 됐지!' 라는 생각도 들었다. 앞으로 Spring 구조에 대하여 더 공부해야겠다.

---
