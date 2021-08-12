UsernamePasswordAuthenticationFilter
=====================================
![image](https://user-images.githubusercontent.com/50267433/129195153-19ea8143-277c-4d64-a0ba-14ad7544c1fc.png)

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
