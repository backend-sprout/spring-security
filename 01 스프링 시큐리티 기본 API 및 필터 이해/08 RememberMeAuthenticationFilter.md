RememberMeAuthenticationFilter
===============================
# RememberMeAuthenticationFilter란      
![image](https://user-images.githubusercontent.com/50267433/129193728-69784562-30b9-4ddf-baec-4378fae8b473.png)   

`세션 만료`, `브라우저 종료로 인한 세션 끊김`, `세션 유실`등으로 **인증 정보를 찾지 못하는 경우가 있다.**      
이로인해 새로운 페이지로 이동시에 **인증 정보를 찾지 못하게 되어 로그 아웃처리가 되는 문제가 발생한다.**       
  
**RememberMeAuthenticationFilter** 는 이러한 문제를 해결하기 위해 존재하는 필터로         
**인증을 시도하고 인증 정보를 받아 사용자가 로그인 상태를 유지시켜주는 작업을 진행한다.**                 

# 조건 
`RememberMeAuthenticationFilter`가 동작하기 위해서는 아래 2가지 조건을 충족해야한다.   
  
* **Authentication 이 Null일 경우(SecurityContext 안에 존재하는 Authentication를 못 찾았을 경우)**           
* **Rember-Me 기능을 활성화 해서 RememberMe Cookie를 발급 받아 보관하고 있는 경우**       
   
Authentication 이 Null 일 경우에 필터가 동작하는 것인데       
Authentication 이 Null이 아닌 경우는 이미 사용자 인증 정보를 받았기 때문이다.        

# 흐름
> 세션 정보를 유실했고 RememberMe 쿠키를 가지고 있는 사용자가 요청을 보냈다 가정하자.  

RememberMeAuthenticationFilter는 동작을 하면서 **RememberMeService 구현체**를 불러와서 사용한다.   
RememberMeService 구현체로는 2가지가 있는데 설명은 아래와 같다.  

* TokenBasedRememberMeServices : 메모리에 RememberMe 쿠키(토큰)를 저장한 경우에 사용     
* PersistenceTokenBasedRememberMeServices : DB에 RememberMe 쿠키(토큰) 저장한 경우에 사용     

각각의 RememberMeService 구현체들은 RememberMe 쿠키(토큰)를 가져와 사용자 RememberMe 쿠키(토큰)와 비교한다.   








