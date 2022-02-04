# DelegationFilterProxy
      
Filter는 Servlet 스펙에 정의된 기술로서         
**Servlet 기반의 웹 애플리케이션들은 ServletFilter를 사용할 수 있다.**          

**Servlet Filter의 장점**
* 요청에 대해서 Servlet 이 처리하기전에 ServletFilter가 전처리 가능하다.    
* 응답에 대해서 Servlet 으로 부터 반환된 값을 Filter가 후처리 가능하다.             
    
<img width="1078" alt="스크린샷 2022-01-25 오후 4 28 28" src="https://user-images.githubusercontent.com/50267433/150930780-fa0adc81-573e-4d9f-bfb3-51a0b656d2fd.png">
  
그러나, 한가지 유념할 점은 ServletFilter는 Servlet 스펙이기에 ServletContainer 에서 생성되고 실행된다.        
즉, 기존에 우리가 사용하는 Spring과 SpringContainer와 별개의 영역에서 사용되고 관리된다는 점이다.     
이는 곧, 서로의 영역에 존재하는 빈을 가져와서 사용할 수 없다는 점이기도하다.(서블릿은 스프링 기능 제공 못받음)  
  
**그런데 Spring Security에서 Filter 클래스를 사용하던 것 같은데?🤔**       
* Spring Security는 사용자가 요청한 모든 요청에 대해서 Filter 기반을 인증/인가 처리하고 있다.      
* 그리고 이 과정에서 Spring의 빈을 주입받고 Spring에서 사용하는 기술들을 사용할 수 있었다.   
   
다시 말하지만, Filter는 Servlet 스펙이기에 원래는 Spring 에서 사용이 쉽지않다.       
그렇기에 별다른 작업 없이 Filter를 Spring Bean 으로 등록한다면 Spring 환경에서 사용되지 않을 것이다.  
     
**그렇다면 어떻게 해결해야할까?🤔**   
* ServletFilter 에서 요청을 받는 것은 변경할 수 없는 작업이다.       
* 그렇기에 ServletFilter에서 Bean으로 만든 Filter로 요청 흐름을 넘길수 있는 구조를 만들어야한다.      
* 바로 이러한 문제를 해결하기 위해 **DelegatingFilterProxy 를 사용하여 요청을 위임하면 된다.**      
           
**💡 DelegatingFilterProxy**           
* **Servlet 스펙의 클래스**이다.                  
* 요청을 받아서 이를 **다른 프레임워크의 Filter 에게 위임할 수 있는 기능을 가지고 있다.**            
* 즉, DelegatingFilterProxy 을 통해 요청이 위임되고 Spring Security의 보안 로직이 동작하는 것이다.     
  
**정리**     
1. 서블릿 필터는 스프링에서 정의된 빈을 주입해서 사용할 수 없다.        
2. DelegatingFilterProxy는 특정한 이름을 가진 스프링 빈을 찾아 그 빈에게 요청을 위임할 수 있다.    
    * **DelegatingFilterProxy 자체는 실제 보안처리를 하지 않는다.**        
    * **springSecurityFilterChain 이름으로 생성된 빈을 ApplicationContext에서 찾아 요청을 위임한다.**      
        * |빈 이름|객체|
          |-----|----|
          |springSecurityFilterChain|필터객체|


# FilterChainProxy 

![image](https://user-images.githubusercontent.com/50267433/152535873-18a0a780-9c38-4f5d-896c-7df549669429.png)

1. springSecurityFilterChain 의 이름으로 생성되는 필터 빈이다.     
2. DelegatingFilterProxy로부터 요청을 위임받고 실제 보안처리를 한다.      
3. Spring Security 초기화시 생성되는 필터들을 관리하고 제어한다.  
    * 스프링 시큐리티가 기본적으로 생성하는 필터
    * 설정 클래스에서 API 추가시 생성되는 필터
4. 사용자의 요청을 필터 순서대로 호출하여 전달한다.  
5. 사용자 정의 필터를 생성해서, 기존의 필터 전/후로 추가할 수 있다.  
    * 필터의 순서를 잘 정의 
6. 마지막 필터까지 인증 및 인가 예외가 발생하지 않으면 보안을 통과한다.  
  
실은 각각의 필터로 넘어갈 때마다, 실제 그 필터로 가는 것이 아니라      
FilterChainProxy를 거치고 이후 필터로 이동하는 구조라는 것을 기억하자    







