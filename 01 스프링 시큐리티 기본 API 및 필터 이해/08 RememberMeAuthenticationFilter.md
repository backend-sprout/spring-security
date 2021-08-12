RememberMeAuthenticationFilter
===============================
![image](https://user-images.githubusercontent.com/50267433/129193728-69784562-30b9-4ddf-baec-4378fae8b473.png)   

# 📘 RememberMeAuthenticationFilter    

`세션 만료`, `브라우저 종료로 인한 세션 끊김`, `세션 유실`등으로 **인증 정보를 찾지 못하는 경우가 있다.**      
이로인해 새로운 페이지로 이동시에 **인증 정보를 찾지 못하게 되어 로그 아웃처리가 되는 문제가 발생한다.**       
  
**RememberMeAuthenticationFilter** 는 이러한 문제를 해결하기 위해 존재하는 필터로         
**인증을 시도하고 인증 정보를 받아 사용자가 로그인 상태를 유지시켜주는 작업을 진행한다.**                 

## 📖 조건 
`RememberMeAuthenticationFilter`가 동작하기 위해서는 아래 2가지 조건을 충족해야한다.   
  
* **Authentication 이 Null일 경우(SecurityContext 안에 존재하는 Authentication를 못 찾았을 경우)**           
* **Rember-Me 기능을 활성화 해서 RememberMe Cookie를 발급 받아 보관하고 있는 경우**       
   
Authentication 이 Null 일 경우에 필터가 동작하는 것인데       
Authentication 이 Null이 아닌 경우는 이미 사용자 인증 정보를 받았기 때문이다.        

# 📗 RememberMeService   
> 세션 정보를 유실했고 RememberMe 쿠키를 가지고 있는 사용자가 요청을 보냈다 가정하자.  

RememberMeAuthenticationFilter는 **RememberMeService 구현체**를 불러와서 사용한다.   
RememberMeService 구현체로는 2가지가 있는데 아래 설명을 보면 된다.    

* **TokenBasedRememberMeServices :** **메모리**에 RememberMe 쿠키(토큰)를 저장한 경우에 사용(기본 14일)     
* **PersistenceTokenBasedRememberMeServices :** **DB**에 RememberMe 쿠키(토큰) 저장한 경우에 사용     

RememberMeService 구현체들은 저장소 및 사용자의 RememberMe 쿠키(토큰)와 비교하는 작업을 진행한다.  

## 📖 Token Cookie 추출   
사용자가 지금 가지고 있는 토큰이 `RememberMe`인지 파악한다.      
`RememberMe`가 아닐 경우 `chain.doFilter()`로 다음 필터에게 요청 흐름을 넘긴다.       

## 📖 토큰 검증  
### 📄 Decode Token     
`RememberMe 토큰`는 특정한 형식을 갖추고 있다.     
그렇기에 `RememberMe 토큰`에 맞는 형식인지 검증한다.         
만약, 형식이 맞지 않는다면 Exception을 발생시킨다.      
 
### 📄 양측 Token 비교   
양측의 토큰을 비교해서 일치하지 않으면 Exception을 발생시킨다.  

### 📄 User 계정 존재 여부 파악   
Token에 들어있는 사용자 정보를 통해서 메모리/DB에서 User 정보를 조회해본다.     
만약, User 정보가 없다면 Exception을 발생시킨다.      

## 📕 새로운 Authentication 생성   
**RemeberMeAuthentication** 이라는 새로운 인증 정보 객체를 생성한다.          
그리고 RemeberMeAuthentication를 AuthenticationManager에게 전달하여      
기존 인증 정보 프로세스가 잘 동작하도록 한다.        


















