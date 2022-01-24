Form Login 인증
================  

# Form 인증 

![image](https://user-images.githubusercontent.com/50267433/129194538-95ed00ec-27b7-4dda-8a17-5fa1d4e0946f.png)


* 사용자는 특정 URL로 데이터(JSON or HTML)를 요청한다.     
* 사용자가 인증이 되지 않았다면, `/login`으로 리다이렉트 한다.(커스텀 없을시, 디폴트 페이지를 넘겨준다.)    
* 사용자가 로그인 요청을 보내면 
* 서버는 로그인 정보에 대한 인증토큰을 만들고 이를 세션에 저장한다.(SecurityContextHolder)   
* 서버는 사용자에게 인증토큰을 내려주며, 사용자는 이후부터 해당 토큰을 통해 인증/인가 작업을 처리받는다.   
    
세션이 나와서 조금 의아하지만, 이 과정은 Form 인증 과정이다.(서버와 클라이언트가 분리 안된 구조)       
그렇기에 세션을 통해서 인증정보를 저장하고 있다.(SecuritContextHolder 를 세션 기준으로 사용했을 것이다.)     

# formLogin

```java
        http
                .formLogin()
                .loginPage("/loginPage")
                .defaultSuccessUrl("/")
                .failureUrl("/login")
                .usernameParameter("userId")
                .passwordParameter("passwd")
                .loginProcessingUrl("/login_proc")
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(final HttpServletRequest request, final HttpServletResponse response, final Authentication authentication) throws IOException, ServletException {
                        System.out.println("authentication = " + authentication.getName());
                        response.sendRedirect("/");
                    }
                })
                .failureHandler(new AuthenticationFailureHandler() {
                    @Override
                    public void onAuthenticationFailure(final HttpServletRequest request, final HttpServletResponse response, final AuthenticationException exception) throws IOException, ServletException {
                        System.out.println("exception = " + exception.getMessage());
                        response.sendRedirect("/login");
                    }
                })
                .permitAll();
```

`http.formLogin()` 이후에 사용할 수있는 메서드들에 대해서 설명한다.
 
|메서드|설명|  
|----|----|  
|`.loginPage("/login.html")`|사용자 정의 로그인| 
|`.defaultSuccessUrl("/home")`|로그인 성공 후 이동 페이지|  
|`.failureUrl("/login.html?error=true")`|로그인 실패 후 이동 페이지|   
|`.usernameParameter("username")`|아이디 파라미터 설정|  
|`.passwordParameter("password")`|패스워드 파라미터 설정|  
|`.loginProcessingUrl("/login")`|로그인 Form Action Url|   
|`.successHandler(new SomethingLoginSucessHandler())`|로그인 성공 후 핸들러|   
|`.failureHandler(new LoginFailureHandler())`|로그인 실패 후 핸들러|   


# Http Basic  

![image](https://user-images.githubusercontent.com/50267433/129194336-2a67d292-249e-4ccc-9348-81b1ca79d1f0.png)



