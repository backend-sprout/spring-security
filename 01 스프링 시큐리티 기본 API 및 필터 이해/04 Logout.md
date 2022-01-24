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



