UsernamePasswordAuthenticationFilter
=====================================
![image](https://user-images.githubusercontent.com/50267433/129195153-19ea8143-277c-4d64-a0ba-14ad7544c1fc.png)

# 코드 
```java
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
        // 구현체마다 알아서 가져온다 -> Filter구현체가 연관된 Provider 구현체 호출 로직 사용
	// provider는 request/response 로부터 ID/Password/토큰 과 같은 정보를 토대로 SecutiryContext 조회
	// 알맞는 값이 있으면 User 와 Autheority 를 담은 Authentication 반환 없으면 Null 반환 
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
