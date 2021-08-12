RememberMeAuthenticationFilter
===============================
    
![image](https://user-images.githubusercontent.com/50267433/129193728-69784562-30b9-4ddf-baec-4378fae8b473.png)   

`세션 만료`, `브라우저 종료로 인한 세션 끊김`, `세션 유실`등으로 **인증 정보를 찾지 못하는 경우가 있다.**      
이로인해 새로운 페이지로 이동시에 **인증 정보를 찾지 못하게 되어 로그 아웃처리가 되는 문제가 발생한다.**       

`RememberMeAuthenticationFilter`는 이러한 문제를 해결하기 위해 존재하는 필터로 
`RememberMeAuthenticationFilter`가 인증을 시도하고 인증 정보를 받아 로그인 상태를 유지시켜준다.         

단, `RememberMeAuthenticationFilter`가 동작하기 위해서는 아래 2가지 조건을 충족해야한다.    
  
* Authentication 이 Null일 경우(SecurityContext 안에 존재하는 Authentication를 못 찾았을 경우)     
* Rember-Me 기능을 활성화 해서 RememberMe Cookie를 발급 받아 보관하고 있는 경우 

  
즉, Authentication 이 Null 일 경우에 필터가 동작하는 것인데       
Authentication 이 Null이 아닌 경우는 이미 사용자 인증 정보를 받았기 때문이다.        




