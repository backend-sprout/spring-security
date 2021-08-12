AnonymousAuthenticationFilter
===============================
![image](https://user-images.githubusercontent.com/50267433/129204106-4afc0a7c-9001-480e-9a46-19f623705d14.png)


* 익명사용자 인증 처리 필터 
* 익명사용자와 인증 사용자를 구분해서 처리하기 위한 용도로 사용  
* 화면에서 인증 여부를 구현할 때 isAnonymous()와 isAuthenticated()로 구분해서 사용
* 인증객체를 세션에 저장하지 않는다.   
  
# AnonymousAuthenticationFilter  
인증 과정은 주로 아래와 같다.   

1. 인증 진행
2. 세션에 User 객체 저장 
3. 자원 접근 
4. 인증 정보 확인  
    * 확인 완료 : Authentication(User) 반환, 접근 허용   
    * **확인 불가 : Authentication(User) Null 반환, 접근 비허용**   
   
일반적인 필터의 경우 인증을 받지 않은 사용자는 Null 로 처리한다.       
그러나 `AnonymousAuthenticationFilter`는 Null 대신 별도의 **익명 사용자 인증 객체를 만들어 처리한다.**       


```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
    if (SecurityContextHolder.getContext().getAuthentication() == null) {
        SecurityContextHolder.getContext().setAuthentication(createAuthentication((HttpServletRequest) req));
        if (logger.isDebugEnabled()) {
            logger.debug("Populated SecurityContextHolder with anonymous token: '"
                + SecurityContextHolder.getContext().getAuthentication() + "'");
			}
    } else {
        if (logger.isDebugEnabled()) {
            logger.debug("SecurityContextHolder not populated with anonymous token, as it already contained: '"
                + SecurityContextHolder.getContext().getAuthentication() + "'");
        }
    }
    chain.doFilter(req, res);
}

protected Authentication createAuthentication(HttpServletRequest request) {
    AnonymousAuthenticationToken auth = new AnonymousAuthenticationToken(key, principal, authorities);
    auth.setDetails(authenticationDetailsSource.buildDetails(request));
    return auth;
}   
```
다른 필터들과 달리 코드를 보면 알 수 있듯이,     
null 일 경우 return 이 아니라 `AnonymousAuthenticationToken`생성 및 리턴을 하고 있다.           
이후 `SecurityContextHolder.getContext().setAuthentication()`처럼 시큐리티 컨텍스트에 저장을 한다.          
참고로, 실제 인증 객체가 아니므로 인증 객체를 `세션`에 저장하지는 않는다.(세션 비어있다.)    
  
시큐리티는 이후 `isAnonymous()`와 `isAuthenticated()`로 `익명`인지 `인증`인지 구분해서 사용한다.       

* 익명 : Login 페이지 보여주기 
* 인증 : Logout 페이지 보여주기  

## chain.doFilter(req, res); 이후 
`chain.doFilter(req, res);`로 이어지다가 마지막으로 실행되는 클래스는 `AbstractSecurityInterceptor`이다.       
`AbstractSecurityInterceptor`의 `--Invocation()`관련 메서드 로직을 보면 아래와 같다.     
   
```java
if (SecurityContextHolder.getContext().getAuthentication() == null) {
    credentialsNotFound(messages.getMessage(
        "AbstractSecurityInterceptor.authenticationNotFound",
        "An Authentication object was not found in the SecurityContext"),
        object, attributes);
}
```
즉, 시큐리티 컨텍스트에서 Authentication 객체를 꺼내서 null인지 아닌지를 검증한다.   
그리고 `AnonymousAuthenticationToken`는 이러한 로직을 수행하기 위해서 존재한다고 이해하면 된다.   

```java
private Authentication authenticateIfRequired() {
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
    if (authentication.isAuthenticated() && !alwaysReauthenticate) {
        if (logger.isDebugEnabled()) {
	    logger.debug("Previously Authenticated: " + authentication);
        }
        return authentication;
    }
    authentication = authenticationManager.authenticate(authentication);
    if (logger.isDebugEnabled()) {
        logger.debug("Successfully Authenticated: " + authentication);
    }
    SecurityContextHolder.getContext().setAuthentication(authentication);
    return authentication;
}
```
그리고 `authenticateIfRequired()`를 실행하면서 `isAuthenticated()`를 실행하는데      
여기서 익명 Authentication인지 아닌지 검증하고 이에 알맞는 로직을 수행한다.       

```java
try {
    this.accessDecisionManager.decide(authenticated, object, attributes);
}
```
이후 위 코드를 통해 `인가` 검증 로직을 수행한다.(해단 Path 접근 가능한지)           
그러나 `익명 Authentication`은 인가 로직을 수행할 수 없기에 `ExceptionTranslationFilter`을 호출하게 된다.   

```java
else if (exception instanceof AccessDeniedException) {
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();    
    if (authenticationTrustResolver.isAnonymous(authentication) || authenticationTrustResolver.isRememberMe(authentication)) {
```
```java
public boolean isAnonymous(Authentication authentication) {
    if ((anonymousClass == null) || (authentication == null)) {
        return false;
    }
    return anonymousClass.isAssignableFrom(authentication.getClass());
}

public boolean isRememberMe(Authentication authentication) {
    if ((rememberMeClass == null) || (authentication == null)) {
        return false;
    }
    return rememberMeClass.isAssignableFrom(authentication.getClass());
}
```
`ExceptionTranslationFilter`의 `handleSpringSecurityException()`를 실행하게 되는데         
`AuthenticationTrustResolver`의 구현체인 `AuthenticationTrustResolverImpl`의 `isAnonymous`를 호출하게 된다.       
이를 통해 `익명/리멤버`인지 검증을 하고 이에 알맞는 로직을 수행한다.      
