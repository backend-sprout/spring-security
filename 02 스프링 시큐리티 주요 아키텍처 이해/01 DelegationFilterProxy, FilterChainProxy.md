# DelegationFilterProxy

Servlet 필터는 Servlet 스펙에 정의된 기술로서   
Servlet 기반의 웹 애플리케이션들은 ServletFilter를 사용할 수 있다.     

요청에 대해서 Servlet 이 처리하기전에 ServletFilter 가 전처리를 해주고     
응답에 대해서도 Servlet 으로 반환된 값을 Filter가 후처리를 해줄 수도 있다.     
   
<img width="1078" alt="스크린샷 2022-01-25 오후 4 28 28" src="https://user-images.githubusercontent.com/50267433/150930780-fa0adc81-573e-4d9f-bfb3-51a0b656d2fd.png">
  
그러나 ServletFilter 는 Servlet 스펙이며 이를 지원하는 ServletContainer 에서 생성되고 실행된다.     
반대로 Spring Bean 들은 Spring Container 에서 동작하기에    
**Servlet Filter 는 Spring 에서 정의된 빈을 주입해서 사용할 수 없다는 문제가 발생한다.**    
    
즉, ServletFilter 에서 Spring Bean을 주입 받을 수 없으며, Spring과 관련된 기능을 활용할 수 없다는 뜻이기도 하다.       
    
그러나 이전 강의들의 보안처리 과정을 돌이켜보면,        
Spring Security는 사용자가 요청한 모든 요청에 대해서 Filter 기반을 인증/인가 처리하고 있다.      
그리고 이 과정에서 Spring의 빈을 주입받고 Spring에서 사용하는 기술들을 사용할 수 있었다.   
 
Spring Security 에서는 필터 기반으로 보안처리를 해야하기 때문에 빈 형식으로 Filter를 구현했다.        
그런데, 사용자가 요청을 하면 현재는 Servlet 기반으로 동작하기 때문에 그 요청을 ServletFilter로 받게된다.     
즉, Filter를 빈 형식으로 구현해도 Servlet 스펙에서는 바로 사용하지 못한다는 문제가 있다.(Filter가 빈이므로)      

**어떻게 해결해야할까?**   
ServletFilter 에서 요청을 받는 것은 변경할 수 없는 작업이다.     
그렇기에 ServletFilter에서 Bean으로만든 Filter로 요청 흐름을 넘길수 있는 구조를 만들어야한다.    
바로 이러한 문제를 해결하기 위해 DelegatingFilterProxy 를 사용하여 요청을 위임하면 된다.    
   
**DelegatingFilterProxy** 는 Servlet 스펙의 클래스이다.(스프링이 아니다)         
DelegatingFilterProxy 는 요청을 받아서 이를 다른 프레임워크에 위임할 수 있는 기능을 가지고 있다.      
즉, DelegatingFilterProxy 을 통해 요청이 위임되고 우리가 아는 Spring Security의 보안 로직이 동작하는 것이다.   

**정리**   
1. 서블릿 필터는 스프링에서 정의된 빈을 주입해서 사용할 수 없다.      
2. DelegatingFilterProxy는 특정한 이름을 가진 스프링 빈을 찾아 그 빈에게 요청을 위임할 수 있다.       
    * springSecurityFilterChain 이름으로 생성된 빈을 ApplicationContext에서 찾아 요청을 위임한다.    
    * DelegatingFilterProxy 자체는 실제 보안처리를 하지 않는다.     
  
# FilterChainProxy 

1.  
2.  ads
3.
4.  
5.
6.
# FilterChainProxy  

