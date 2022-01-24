# RememberMe
# RememberMe 인증 

1. 세션이 만료되고 웹 브라우저가 종료된 후에도 애플리케이션이 사용자를 기억하는 기능   
2. Remember-Me 쿠키에 대한 Http 요청을 확인한 후  
   토큰 기반 인증을 사용해 유효성을 검사하고 토큰이 검증되면 사용자는 로그인된다.       
3. 사용자 라이프 사이클 
   * 인증 성공(Remember-Me) 쿠키 설정   
   * 인증 실패(쿠키가 존재하면 쿠키 무효화)     
   * 로그아웃(쿠키가 존재하면 쿠키 무효화)        

# rememberMe() 


```java
        http
                .rememberMe()
                .rememberMeParameter("remember")
                .tokenValiditySeconds(3600)
                .alwaysRemember(false)
                .userDetailsService(userDetailsService());
```

* 리멤버미 기능을 활성화시킨다.   

|메서드|설명|
|-----|--|
|`rememberMeParameter("rememberMe")`|기본 파라미터명은 remember-me|
|`tokenValiditySeconds(3600)`|Default는 14일|
|`alwaysRemember(true)`|리멤버 미 기능이 활성화 되지 않아도 항상 실행|
|`userDetailsService(new SomethingUserDetailsService())`|리멤버미 기능을 수행할때, 시스템에 있는 사용자의 계저 조회시 사용|   
