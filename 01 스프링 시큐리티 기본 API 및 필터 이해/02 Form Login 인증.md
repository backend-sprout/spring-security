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
  
![image](https://user-images.githubusercontent.com/50267433/129194774-34a48278-77a8-48a8-976d-1e325e9c746e.png)


# Http Basic  

![image](https://user-images.githubusercontent.com/50267433/129194336-2a67d292-249e-4ccc-9348-81b1ca79d1f0.png)



