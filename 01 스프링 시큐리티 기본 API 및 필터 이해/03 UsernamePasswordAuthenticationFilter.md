UsernamePasswordAuthenticationFilter
=====================================
![image](https://user-images.githubusercontent.com/50267433/129195153-19ea8143-277c-4d64-a0ba-14ad7544c1fc.png)


* FormLogin 적용시, 적용되는 필터는 UsernamePasswordAuthenticationFilter 다.   
* 필터의 `AntPathRequestMatcher(/login)`을 통해 요청 정보가 `로그인 프로세싱 URL`과 매칭되는지 확인한다.     
    * 정확히 말하면, `AbstractAuthenticationProcessingFilter` 의    
      `doFilter()` 과정 중에서 `requiresAuthentication()`를 통해 매칭 검증을 한다. 
    * 매칭되지 않았다면 다음 필터로 작업을 넘긴다.   
* 요청 정보의 데이터를 활용하여 Authentication 구현체를 생성한다.
    * ```java
      UsernamePasswordAuthenticationToken authRequest = 
          new UsernamePasswordAuthenticationToken(username, password);
      ```  
* AuthenticationManager 구현체의 authenticate()를 호출하고 인자값으로 Authentication 구현체를 사용한다.   
    * authenticate() 는 Authentication 구현체의 값이 올바른지 **인증 검증 작업을 진행한다.**   
    * 즉, 해당 메서드를 통해 인증의 성공 및 실패가 결정된다.    
    * ```java
      this.getAuthenticationManager().authenticate(authRequest);
      ``` 
* authenticate()가 실행되면서 내부적으로 AuthenticationProvider 에게 인증 검증 작업을 위임한다.  
    * 인증에서 실패하면, AuthenticationException 을 발생하고 AbstractAuthenticationProcessingFilter에서 catch 한다.    
* AuthenticationProvider가 인증에 성공하게 되면, Authentication 구현체를 만들고 반환한다.      
    * 인증에 성공한 정보(User 정보) + 인가 정보를 가지고 있는 Authentication 구현체이다.    
* AuthenticationManager 는 전달받은 Authentication 구현체를 필터에 반환한다.  
* 필터는 SecurityContext 에 저장한다.(전역적으로 사용가능)       
* 이후 SuccessHandler를 호출한다.  

# 코드 
```java

// 구현체마다 알아서 가져온다 -> Filter 구현체는 request로부터 ID/PASSWORD 를 추출 및 이를 통한 Authentication 객체 생성   
// Filter 구현체 연관된 AuthenticationManger 구현체(ProviderManager)의 authenticate()을 호출하고 생성한 Authentication도 같이 넘긴다.  
// AuthenticationManger 구현체(ProviderManager)는 여러 Provider 중 사용 가능한 Provider를 찾는다.(AbstractUserDetailsAuthenticationProvider)    
// AuthenticationManger 구현체(ProviderManager)가 provider를 찾으면 authenticate()를 호출하면서 Authentication 객체도 넘긴다.     
// provider(AbstractUserDetailsAuthenticationProvider)의 authenticate()이 성공하면 User + Authorities = Authentication 생성 및 반환 실패시 Null 반환 
	
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
    HttpServletRequest request = (HttpServletRequest) req;
    HttpServletResponse response = (HttpServletResponse) res;
    
    // url 맞는지 확인 
    if (!requiresAuthentication(request, response)) {
        chain.doFilter(request, response);
	return;
    }
    if (logger.isDebugEnabled()) {
        logger.debug("Request is to process authentication");
    }
    // 인증 객체 준비 
    Authentication authResult;
    try {
        authResult = attemptAuthentication(request, response);
        if (authResult == null) {
	    return;
        }
        sessionStrategy.onAuthentication(authResult, request, response);
    } catch (InternalAuthenticationServiceException failed) {
        logger.error("An internal error occurred while trying to authenticate the user.", failed);
	unsuccessfulAuthentication(request, response, failed);
	return;
    } catch (AuthenticationException failed) {
        unsuccessfulAuthentication(request, response, failed);
        return;
    }
    if (continueChainBeforeSuccessfulAuthentication) {
        chain.doFilter(request, response);
    }
    successfulAuthentication(request, response, chain, +authResult);
}
```
```java
public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
    if (postOnly && !request.getMethod().equals("POST")) { 
        throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
    }
    String username = obtainUsername(request);
    String password = obtainPassword(request);
    if (username == null) {
        username = "";
    }
    if (password == null) {
        password = "";
    }
    username = username.trim();
    UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
    // Allow subclasses to set the "details" property
    setDetails(request, authRequest);

    return this.getAuthenticationManager().authenticate(authRequest);
}
```
