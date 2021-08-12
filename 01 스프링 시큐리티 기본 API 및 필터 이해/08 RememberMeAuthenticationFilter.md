RememberMeAuthenticationFilter
===============================
    
![image](https://user-images.githubusercontent.com/50267433/129193728-69784562-30b9-4ddf-baec-4378fae8b473.png)   

RememberMeAuthenticationFilter는 사용자 요청을 받아서 그 요청을 처리하는 조건이 있다.   
  
* Authentication 이 Null일 경우(SecurityContext 안에 존재하는 Authentication를 못 찾았을 경우)     
* 세션 만료, 브라우저 종료로 인한 세션 끊김, 세션 유실등등    
  
즉, Authentication 이 Null 일 경우에 필터가 동작하는 것인데       
Authentication 이 Null이 아닌 경우는 이미 사용자 인증 정보를 받았기 때문이다.        

그래서 `RememberMeAuthenticationFilter`가 인증을 시도하고 인증 정보를 받아 로그인 상태를 유지시켜준다.          



