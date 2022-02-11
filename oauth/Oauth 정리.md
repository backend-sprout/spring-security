# Oauth 정리.md
> OAuth 에 대한 내용보다는 Spring 에서 어떤 기능들을 제공하는지 정리하는 문서
> 

- `spring-security-oauth2-authorization-server:0.2.1` 기준
- 보다 자세한 내용은 해당 [docs](https://projects.spring.io/spring-security-oauth/docs/oauth2.html) 로

# 1. Authorization Server

## 1-1. 최소한의 기본 조건

1. 관련 의존성들 추가 
2. `@EnableAuthorizationServer` 어노테이션을 붙여야 한다. 
3. 하나 이상의 클라이언트 ID / Secret 을 지정해야한다.  

```java
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'org.springframework.security:spring-security-oauth2-authorization-server:0.2.1'
```

```java
security:
  oauth2:
    client:
      client-id: first-client
      client-secret: noonewilleverguess
```

위와 같이 지정하고, AutoConfiguration 설정이 아니기에

AuthorizationConfig 클래스에서 yml로 부터 받아들여 활용하는 방식으로 진행한다. 

## 1-2. AutoConfiguration Off (디폴트 값 제거)

`**RegisteredClientRepository` 의 기본 설정** 

- 기본 패스워드 정책 : NoOpPasswordEncoder
- 기본 인증 방법
    - `authorization_code`
    - `password`
    - `client_credentials`
    - `implicit`
    - `refresh_token`

**새로운 Bean 주입으로 설정 대체** 

- `AuthenticationManager`: 실제 사용자 데이터를 조회하고 검증
- `TokenStore`: 발급된 토큰 정보들을 저장
- `AccessTokenConverter`: AccessToken을 다른 형태로 변환(JWT)

## 1-3. `AuthorizationServerConfig` 버전 0.1.2 이전

```java
@Configuration
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired DataSource dataSource;

    protected void configure(ClientDetailsServiceConfigurer clients) {
        clients
            .jdbc(this.dataSource)
            .passwordEncoder(PasswordEncoderFactories.createDelegatingPasswordEncoder());
    }
}
```

현재는, `AuthorizationServerConfigurerAdapter`에 대해서 import 도 불가하다.   

## 1-4. `AuthorizationServerConfig` 버전 0.2 이후

```java
@Configuration(proxyBeanMethods = false)
public class AuthorizationServerConfig {
    @Bean
    public RegisteredClientRepository registeredClientRepository() {
        RegisteredClient registeredClient = RegisteredClient.withId(UUID.randomUUID().toString())
          .clientId("articles-client")
          .clientSecret("{noop}secret")
          .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
          .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
          .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
          .redirectUri("http://127.0.0.1:8080/login/oauth2/code/articles-client-oidc")
          .redirectUri("http://127.0.0.1:8080/authorized")
          .scope(OidcScopes.OPENID)
          .scope("articles.read")
          .build();
        return new InMemoryRegisteredClientRepository(registeredClient);
    }

    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SecurityFilterChain authServerSecurityFilterChain(HttpSecurity http) throws Exception {
        OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
        return http.formLogin(Customizer.withDefaults()).build();
    }

		@Bean
		public JWKSource<SecurityContext> jwkSource() {
		    RSAKey rsaKey = generateRsa();
		    JWKSet jwkSet = new JWKSet(rsaKey);
		    return (jwkSelector, securityContext) -> jwkSelector.select(jwkSet);
		}
		
		private static RSAKey generateRsa() {
		    KeyPair keyPair = generateRsaKey();
		    RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
		    RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
		    return new RSAKey.Builder(publicKey)
		      .privateKey(privateKey)
		      .keyID(UUID.randomUUID().toString())
		      .build();
		}
		
		private static KeyPair generateRsaKey() {
		    KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
		    keyPairGenerator.initialize(2048);
		    return keyPairGenerator.generateKeyPair();
		}

		@Bean
		public ProviderSettings providerSettings() {
		    return ProviderSettings.builder()
		      .issuer("http://auth-server:9000")
		      .build();
		}
}
```

## 1-5. AuthenticationManager 적용 방법 및 시기

### 1-5-1. Spring Security Authentication Architecture

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/729faba3-5a77-4df1-9be5-5386b729a8cb/Untitled.png)

기본적인 Spring Security 필터 및 인증 과정은 위와 같다.

### 1-5-2. AuthenticationManager 란?

AuthenticationManager 는 인증을 처리하는 실질적인 주체이다.    

- AuthenticationManager 는 내부에서 인증에 적합한 AuthenticationProvider 를 조회하고 사용한다.
- AuthenticationProvider 는 내부에서 사용자 데이터를 조회할 수 있는 UserDetailsService 를 이용한다.
- AuthenticationManager 의 작업의 반환값은 Authentication 타입의 객체이다.(Principal 을 포함하는).

즉, AuthenticationManager 를 적용한다는 것은   

사용자 저장소에 저장된 사용자를 기반으로 인증을 처리할 때 사용한다.(적용은 되지만, Password 가 유용)  

그리고 필요에 따라서 AccessTokenConverter 를 이용해, 사용자 식별자 정보 JWT로 변환도 가능하다.  

### 1-5-3. AuthenticationManager 적용 시기

```java
public interface AuthenticationManager {

	Authentication authenticate(Authentication authentication) throws AuthenticationException;

}
```

- 사용자 정보를 가져오므로 Provider나 UserDetailsService를 의존해서 사용하는 것이 좋다.
- Default로 사용되는 AuthenticationManager 도 있으니, Provider를 추가하는 방식도 고려가능하다. (`ProviderManager` 클래스일 것이다.)

모든 인증 확인 방식에서 AuthenticationManager 가 필요하지는 않다.     

더불어서, 모든 요청 흐름에 대해서도 AuthenticationManager 가 필요하지는 않다.    

**인증 확인**

- Client Credentials : 사용자가 아닌 클라이언트의 인가를 기준으로 토큰을 요청이기에 불필요
- Refresh Token : Refresh Token 의 인가에만 기반한 토큰을 요청이기에 불필요

**요청 흐름**

- Authorization Code : OAuth 토큰을 요청할 때가 아니라 사용자가 로그인 되었을 때 확인
- Implicit : OAuth 토큰을 요청할 때가 아니라 사용자가 로그인 되었을 때 확인
- 추론하자면, 리소스 서버에서 AccessToken 검증시 필요하다는 의미 같음

**정리하자면** 

- **Password : 최종 사용자의 자격 증명을 기반으로 하는 코드를 반환하기에 적용하기 괜찮다.**

### 1-5-4. AuthenticationManager 적용 방법

1. `UserDetailsService` 구현체인 `CustomUserDetails` 를 작성하여 빈으로 등록   
2. `AuthenticationManager` 구현체인 `CustomAuthenticationManager` 를 작성하여 빈으로 등록
3. `AuthorizationServerConfigurerAdapter` 와 [`AuthenticationManager`](https://docs.spring.io/spring-security-oauth2-boot/docs/2.2.x-SNAPSHOT/reference/html/boot-features-security-oauth2-authorization-server.html#oauth2-boot-authorization-server-password-grant-autowired-authentication-manager) 수동으로 의존 연결

### 1-5-4-1. ****`UserDetailsService` 빈으로 등록**

```java
    @Autowired private final DataSource dataSource;

		
    @Bean
    @Override
    public UserDetailsService userDetailsService() {
        return new JdbcUserDetailsManager(this.dataSource);
    }
```

   

- 대략적인 UserDetailsService 을 빈으로 만드는 과정이다
- 기본 `UserDetailsService`는 `InMemoryUserDetailsManager` 로써
인메모리로 유저 정보를 저장하고 관리하는 방식이다.
- 만약, 우리가 특정 DB에 유저 정보를 저장하고 관리한다면 위와 같이 해야한다.

**예시 클래스**

- 예시로 나온 `JdbcUserDetailsManager` 는, `JDBCTemplate` 을 의존받아야 사용이 가능하다.
- 단, 여기서 궁금증은 어떤 테이블에 접근해서 데이터를 가져오는지에 대해서는 나오지 않았다.
- 내부 코드를 보니 `usersByUsernameQuery`를 쿼리로 사용하는데 해당 부분을 주입해줘야 할 것 같다.

### 1-5-4-2 ****`AuthenticationManager` 빈으로 등록**

> 보다 자세히 말하면, ****`AuthenticationProvider`**** 를 빈으로 등록한다.
> 

```java

    @Bean(BeansId.AUTHENTICATION_MANAGER)
    @Override
    public AuthenticationManager authenticationManagerBean() {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) {
        auth.authenticationProvider(customAuthenticationProvider());
    }
```

기존 `AuthenticationManager` 을 다시 올리되, 

`customAuthenticationProvider()` , 즉 `AuthenticationProvider` 를 등록해서 올린다. 

properties 나 yml 을 이용해서 클래스 이름을 지정하는 Auto Configuration 도 가능하다. 

**어디서 사용하지? 🤔**     

일단 필자 생각으로는, 여러 DB 에 접근해서 데이터를 가져올 때 사용할 것 같다.       

`AuthenticationProvider` 에서 각각의 정의된 `UserDetailsService` 에 접근하는 느낌이다.   

## 1-6 0.1.2 이전 버전은 Spring 5.1 에서...

Spring Security 5.1은 JWT 인코딩 JWK 서명 권한 부여만 지원하며     

이전 버전의 Authorization Server 는 JWK Set URI와 함께 제공되지 않습니다.  

하지만, 기본적인 기능은 지원한다.  

만약, 스프링 5.1 과 연동을 하고자 한다면 아래와 같은 요소들을 추가해야한다.  

- Configure it to use JWKs
- Add a JWK Set URI endpoint

**하지만 이외의 장점으로는**  

`/oauth/token` 엔드포인트에 POST하여 토큰을 가져온 다음 
Spring Security 5.1 Resource Server에 제공할 수 있다.  

# JWT로 Converter 하기

참고 : [https://www.baeldung.com/spring-security-oauth-jwt-legacy](https://www.baeldung.com/spring-security-oauth-jwt-legacy)
