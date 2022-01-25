# DelegationFilterProxy

Servlet 필터는 Servlet 스펙에 정의된 기술로서   
Servlet 기반의 웹 애플리케이션들은 ServletFilter를 사용할 수 있다.     

요청에 대해서 Servlet 이 처리하기전에 ServletFilter 가 전처리를 해주고     
응답에 대해서도 Servlet 으로 반환된 값을 Filter가 후처리를 해줄 수도 있다.     
   
<img width="1078" alt="스크린샷 2022-01-25 오후 4 28 28" src="https://user-images.githubusercontent.com/50267433/150930780-fa0adc81-573e-4d9f-bfb3-51a0b656d2fd.png">
  
그러나 ServletFilter 는 Servlet 스펙이며 이를 지원하는 ServletContainer 에서 생성되고 실행된다.     
반대로 Spring Bean 들은 Spring Container 에서 동작하기에    
**Servlet Filter 는 Spring 에서 정의된 빈을 주입해서 사용할 수 없다는 문제가 발생한다.**    
    
즉, ServletFilter 에서 Spring Bean을 주입 받을 수 없으며,    
Spring과 관련된 기능을 활용할 수 없다는 뜻이기도하다.       





# FilterChainProxy  

