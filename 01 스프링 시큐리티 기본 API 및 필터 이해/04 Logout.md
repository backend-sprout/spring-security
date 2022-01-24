# Logout
# 인증 API - LogOut
![image](https://user-images.githubusercontent.com/50267433/150732064-30c61b0f-cd05-4547-b51a-154b03d5587e.png)
  
1. 클라이언트는 로그아웃 URL을 통해 로그아웃 요청을 보낸다.    
2. 서버는 `세션 무효화`, `인증 토큰 삭제`, `쿠키정보 삭제`, `로그인 페이지로 리다이렉트` 작업을 수행한다.  

# logout() 

```java
        http
                .logout()
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login")
                .addLogoutHandler(new LogoutHandler() {
                    @Override
                    public void logout(final HttpServletRequest request, final HttpServletResponse response, final Authentication authentication) {
                        HttpSession session = request.getSession();
                        session.invalidate();
                    }
                })
                .logoutSuccessHandler(new LogoutSuccessHandler() {
                    @Override
                    public void onLogoutSuccess(final HttpServletRequest request, final HttpServletResponse response, final Authentication authentication) throws IOException, ServletException {
                        response.sendRedirect("/login");
                    }
                })
                .deleteCookies("remember-me");
```
* 로그아웃 기능이 작동하도록 한다.   

|메서드|설명|
|----|---|
|`logoutUrl("/logout")`|로그아웃 처리|
|`logoutSuccessUrl("/login")`|로그아웃 처리 URL|
|`deleteCookies("JSESSIONID", "remember-me")`|로그 아웃 후 쿠키 삭제|
|`addLogoutHandler(new SomethingAddLogoutHandler())`|로그아웃 핸들러|
|`logoutSuccessHandler(new SomethingLogoutSuccessHandler())`|로그아웃 성공 후 핸들러|

# LogoutFilter

![image](https://user-images.githubusercontent.com/50267433/150733440-865cf382-6d51-47ad-8fab-2b75843287ff.png)  
![image](https://user-images.githubusercontent.com/50267433/129195839-e05b4bde-f81a-4ce9-89f4-7f98546cf911.png)  
  
**간단 설명**   
1. LogoutFilter 에서 AntPathRequestMatcher를 통해서 Logout URL 인지 매칭을 진행한다. 
2. Logout URL이 아닌 경우, 다음 필터로 요청 흐름을 전송한다.    
3. Logout URL인 경우, SecurityContext로 부터 Authentication 객체를 가져온다.    
4. SecurityContextHandler 에 Authentication 객체를 전달한다.   
5. 이후 필요에따라, `세션 무효화`, `쿠키 삭제`, `SecurityContextHolder.clearContext()`를 사용한다.   
6. 위 작업이 끝나면, SimpleUrlLogoutSuccessHandler를 호출한다.   
