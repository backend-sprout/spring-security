UsernamePasswordAuthenticationFilter
=====================================
![image](https://user-images.githubusercontent.com/50267433/129195153-19ea8143-277c-4d64-a0ba-14ad7544c1fc.png)

**쉽게 정리하면 아래와 같다.**  
1. Filter는 요청 URL이 Login 관련 URL인지 검증한다.   
2. Login 관련 URL 이라면, ID/PASSWORD 를 추출 및 이를 통한 Authentication 객체 생성한다.  
3. AuthenticationManger 구현체의 authenticate()을 호출하고 생성한 Authentication도 같이 넘긴다.   
4. AuthenticationManger 구현체는 여러 Provider 중 사용 가능한 Provider를 찾는다.(타입은 AbstractUserDetailsAuthenticationProvider)    
5. AuthenticationManger 구현체가 provider를 찾으면 authenticate()를 호출하면서 Authentication 객체도 넘긴다.  
6. provider의 authenticate()이 성공하면 User + Authorities = Authentication 생성 및 반환하고 실패시 Null을 반환한다.  
7. 역순으로 해당 Authentication 객체를 필터까지 반환한다.    
8. AuthenticationFilter는 Authentication 객체가 NULL 인지 판단한다.     
9. 반환된 Authentication 객체가 NULL이 아니면, SecurityContext 에 저장한다.   
10. 이후 SuccessHandler를 호출한다.      

**실제 객체 이름 기반은 아래와 같다.**  
1. FormLogin은 UsernamePasswordAuthenticationFilter 를 인증 필터로 사용한다.         
2. UsernamePasswordAuthenticationFilter 는 `requiresAuthentication()`를 통해 URL 검증을한다.   
3. `requiresAuthentication()`내부는 RequestMatcher 구현체인 AntPathRequestMatcher 에게 작업을 위임한다.  
4. AntPathRequestMatcher 는 `match()`를 통해 로그인 URL인지 검증한다.   
5. 로그인 URL 매치이 성공했다면, `attemptAuthentication()`를 통해 인증 검증을 진행한다.    
6. `attemptAuthentication()`에서는 요청 정보에 따라 Authentication의 구현체인  
    UsernamePasswordAuthenticationToken 를 만든다.   
7. 이후, `getAuthenticationManager()`를 통해 AuthenticationManager의 하위 구현체인 ProviderManager 를 가져온다.  
8. ProvierManger 의 `authenticate()`를 통해 인증작업을 하고, 인자 값으로 UsernamePasswordAuthenticationToken를 준다.   
9. ProvierManger 의 `authenticate()`는 사용가능한 Provider 를 목록에서 조회한다.    
10. 사용 가능한 Provier는 `authentication()`를 호출하고 UsernamePasswordAuthenticationToken를 넘긴다. 
11. 내부 메서드에서 인증 과정에 대해서 처리한다.   
12. 실패시 doFilter() 에서 try/catch가 발생한다.   
13. 성공시 새로운 Authentication 구현체를 만드는데, `인증에 성공한 정보(User 정보)` + `인가 정보`를 가지고 만든다.   
14. 생성된 Authentication 구현체는 Filter 까지 반환된다.   
15. Filter는 Null인지 검증하고, 아니라면 SecurityContext에 저장한다.   
16. 이후 SuccessHandler를 호출한다.    

# AbstractAuthenticationProcessingFilter  
```java
public abstract class AbstractAuthenticationProcessingFilter {
    ... // 생략 	
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;
    
        // Login url 맞는지 확인 
        if (!requiresAuthentication(request, response)) {
            chain.doFilter(request, response);
	    return;
        }
        if (logger.isDebugEnabled()) {
            logger.debug("Request is to process authentication");
        }
	
        // 인증된 후의 Authentication 객체 저장하기
	// 인증 안되면 catch에서 처리한다.   
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
    
    ... // 생략 
    
    protected boolean requiresAuthentication(HttpServletRequest request, HttpServletResponse response) {
	if (this.requiresAuthenticationRequestMatcher.matches(request)) {
	    return true;
	}
	if (this.logger.isTraceEnabled()) {
	    this.logger
                .trace(LogMessage.format("Did not match request to %s", this.requiresAuthenticationRequestMatcher));
	}
	    return false;
    }
    
    ... // 생략 
}
```
인증 필터들을 위한 `추상화된 인증 필터 클래스`다.       
단, LoginUrl을 판별하는 방식이 `RequestMatcher requiresAuthenticationRequestMatcher.matches(request)`를 통해서 진행된다.           
이때 구현체로 AntPathRequestMatcher를 사용하는데 SecurityConfig로 설정한 값으로 생성되고 이를 통해 비교 작업을 한다고 추측된다.    

* **만약, session 기반이 아니라면 이에 대해서 어떻게 처리하는지 살펴봐야될 것 같다.**   
