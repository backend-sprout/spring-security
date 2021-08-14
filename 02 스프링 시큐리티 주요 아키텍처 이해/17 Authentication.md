Authentication
==================
# Authentication
> 인증, 인증 주체    

Authentication는 **당신이 누구인지를 증명하는 것이다.**   
* 사용자의 인증 정보를 저장하는 토큰 개념 
* **인증시 :** id와 password를 담고 인증 검증을 위해 전달되어 사용된다.   
* **인증후 :** 최종 인증 결과(user 객체, 권한 정보)를 담고 SecurityContext에 저장되어 전역적으로 참조가 가능하다.   
    * `Authentication authentication = SecurityContextHolder.getContext().getAuthentication();` 

**구조**    
* **principal :** 사용자의 ID 혹은 User 객체를 저장 
* **credentials :** 사용자 비밀번호 
* **authorities :** 인증된 사용자의 권한 목록
* **details :** 인증 부가 정보 
* **Authenticated :** 인증 여부 

## Authentication 흐름  
  
![image](https://user-images.githubusercontent.com/50267433/129443363-6e4e3d0d-b8ff-4774-a7cb-315917ab7975.png)
  
1. UserNamePasswordAuthenticationFilter 는 Request 정보를 받아서 Authentication 객체를 만든다.     
2. AuthenticationManger(Provider) 가 인증 객체를 가지고 인증 처리를 진행한다.     
3. **인증이 성공하면 :** Authentication를 새로 만들면서 authorities 추가 및 authenticated 를 true로 한다.       
   **인증이 실패하면 :** 더 이상 진행을 막고 예외를 발생시킨다.         
4. ThreadLocal 에 존재하는 SecurityContext 안에 저장을 한다.    
5. ThreadLocal 에 존재하기에 전역적으로 사용할 수 있게 된다.   





