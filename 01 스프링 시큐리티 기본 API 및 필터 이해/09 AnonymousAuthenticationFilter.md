AnonymousAuthenticationFilter
===============================
![image](https://user-images.githubusercontent.com/50267433/129204106-4afc0a7c-9001-480e-9a46-19f623705d14.png)

# AnonymousAuthenticationFilter  
인증 과정은 주로 아래와 같다.   

1. 인증 진행
2. 세션에 User 객체 저장 
3. 자원 접근 
4. 인증 정보 확인 
    * 확인 완료 : 접근 허용   
    * **확인 불가 : Null 반환**   
   
일반적인 필터의 경우 인증을 받지 않은 사용자는 Null 로 처리한다.       
그러나 `AnonymousAuthenticationFilter`는 Null 대신 별도의 **익명 사용자 인증 객체를 만들어 처리한다.**       




* 익명사용자 인증 처리 필터 
* 익명사용자와 인증 사용자를 구분해서 처리하기 위한 용도로 사용  
* 화면에서 인증 여부를 구현할 때 isAnonymous()와 isAuthenticated()로 구분해서 사용
* 인증객체를 세션에 저장하지 않는다.   


