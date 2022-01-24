# 권한 설정과 표현식 

**선언적 방식**
* URL : 
    * ```java
      http.antMatchers("/users/**").hasRole("USER")`   
      ```
* Method : 
    * ```java
      @PreAuthroize("hasRole('USER')") 
      public void user() { System.out.println("user") }
      ```
**동적 방식 - DB연동 프로그래밍**     
* URL
* Method 

## 선언적 방식 

```java
        http
                .antMatcher("/shop/**")
                .authorizeRequests()
                    .antMatchers("/shop/login", "/shop/users/**").permitAll()
                    .antMatchers("/shop/mypage").hasRole("USER")
                    .antMatchers("/shop/admin/pay").access("hasRole('ADMIN')")
                    .antMatchers("/shop/admin/**").access("hasRole('ADMIN') or hasRole('SYS')")
                    .anyRequest().authenticated();
```
선언적 방식을 통해 각 URL마다 인가 설정 작업을 진행했다.     
다만, 주의점이 있는데 인가 설정시 구체적인 경로가 먼저 오고 그것보다 큰 범위의 경로가 뒤에 오도록 한다.       
왜냐하면 선언된 순서대로 인가 검증을 진행하기에, 큰 범위를 앞에두면 뒤에는 검증이 이루어지지 않을 수 있다.     
## 표현식 

|메서드|설명|
|----|---|
|`authenticated()`|인증된 사용자의 접근을 허용|
|`fullyAuthenticated()`|인증된 사용자의 접근을 허용, rememberMe 제외|
|`permitAll()`|무조건 접근을 허용|
|`denyAll()`|무조건 접근을 허용하지 않음|
|`anonymous()`|익명사용자의 접근을 허용|
|`rememberMe()`|기억하기를 통해 인증된 사용자의 접근을 허용|
|`access(String)`|주어진 SpEL 표현식의 평가 결과가 true이면 접근 허용|
|`hasRole(String)`|사용자가 주어진 역할이 있다면, 접근을 허용|
|`hasAuthority(String)`|사용자가 주어진 권한이 있다면 접근을 허용|
|`hasAnyRole(Strings..)`|사용자가 주어진 역할 중 어떤것이라도 있다면 접근을 허용|
|`hasAnyAuthroity(Strings..)`|사용자가 주어진 권한 중 어떤것이라도 있다면 접근을 허용|
|`helpAddress(String)`|주어진 IP로부터 요청이 있다면 접근을 허용|


