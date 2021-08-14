Authentication
==================
# Authentication
> 인증, 인증 주체    

Authentication는 **당신이 누구인지를 증명하는 것이다.**   
* 사용자의 인증 정보를 저장하는 토큰 개념 
* **인증시 :** id와 password를 담고 인증 검증을 위해 전달되어 사용된다.   
* **인증후 :** 최종 인증 결과(user 객체, 권한 정보)를 담고 SecurityContext에 저장되어 전역적으로 참조가 가능하다.   
    * `Authentication authentication = SecurityContextHolder.getContext().getAuthentication();` 

**구조**    
* **principal :** 사용자의 ID 혹은 User 객체를 저장 
* **credentials :** 사용자 비밀번호 
* **authorities :** 인증된 사용자의 권한 목록
* **details :** 인증 부가 정보 
* **Authenticated :** 인증 여부 

## Authentication 흐름  
  
![image](https://user-images.githubusercontent.com/50267433/129443363-6e4e3d0d-b8ff-4774-a7cb-315917ab7975.png)
  
1. UserNamePasswordAuthenticationFilter 는 Request 정보를 받아서 Authentication 객체를 만든다.     
2. AuthenticationManger(Provider) 가 인증 객체를 가지고 인증 처리를 진행한다.     
3. **인증이 성공하면 :** Authentication를 새로 만들면서 authorities 추가 및 authenticated 를 true로 한다.       
   **인증이 실패하면 :** 더 이상 진행을 막고 예외를 발생시킨다.         
4. ThreadLocal 에 존재하는 SecurityContext 안에 저장을 한다.    
5. ThreadLocal 에 존재하기에 전역적으로 사용할 수 있게 된다.   

## 코드 

**AbstractAuthenticationProcessingFilter**
```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
    HttpServletRequest request = (HttpServletRequest) req;
    HttpServletResponse response = (HttpServletResponse) res;
    
    if (!requiresAuthentication(request, response)) {
        chain.doFilter(request, response);
	     return;
    }
    if (logger.isDebugEnabled()) {
        logger.debug("Request is to process authentication");
    }
    Authentication authResult;
    try {
        authResult = attemptAuthentication(request, response);          // <----- 여기 -------- 
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
```
UsernamePasswordAuthenticationFilter는 오버 라이딩하지 않은 `doFilter()`    
즉, 상위 클래스인 AbstractAuthenticationProcessingFilter의 doFilter()를 실행하면서         
UsernamePasswordAuthenticationFilter이 오버이딩한 `attemptAuthentication()`를 호출한다.      

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
    setDetails(request, authRequest);
    return this.getAuthenticationManager().authenticate(authRequest);
}
```
이 과정에서 `Authentication`을 상속 받은 `AbstractAuthenticationToken`을 상속받은        
`UsernamePasswordAuthenticationToken` 구현체를 통해 `Authentication`객체를 준비한다.(인증 false)       
이후 `AuthenticationManger`의 구현체인 `x`의 `authenticate()`를 호출하면서 작업을 위임한다.   










