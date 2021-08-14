Authentication
==================
# Authentication
> 인증, 인증 주체    

Authentication는 **당신이 누구인지를 증명하는 것이다.**   
* 사용자의 인증 정보를 저장하는 토큰 개념 
* **인증시 :** id와 password를 담고 인증 검증을 위해 전달되어 사용된다.   
* **인증후 :** 최종 인증 결과(user 객체, 권한 정보)를 담고 SecurityContext에 저장되어 전역적으로 참조가 가능하다.   
    * Authentication authentication = SecurityContextHolder.getContext().getAuthentication(); 

**구조**    
* **principal :** 사용자의 ID 혹은 User 객체를 저장 
* **credentials :** 사용자 비밀번호 
* **authorities :** 인증된 사용자의 권한 목록
* **details :** 인증 부가 정보 
* **Authenticated :** 인증 여부 
* 



