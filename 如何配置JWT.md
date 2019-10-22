# 配置 Spring Security JWT

### 一、配置pom.xnl依赖

配置jwt.version，具体版本号已官网为准

	<properties>
    	<jwt.version>0.9.1</jwt.version>
		<problem.version>0.23.0</problem.version>
	</properties>

配置jwt依赖

    <dependency>
	    <groupId>io.jsonwebtoken</groupId>
	    <artifactId>jjwt</artifactId>
	    <version>${jwt.version}</version>
    </dependency>

配置Spring Security依赖
    
    <dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

配置problem-spring-web依赖(可选)

    <dependency>
	    <groupId>org.zalando</groupId>
	    <artifactId>problem-spring-web</artifactId>
	    <version>${problem.version}</version>
    </dependency>
如果选择problem-spring-web需要进行如下配置

1、新增SecurityExceptionHandler.java 对象，否则problem-spring-web不会对异常进行包装

2、配置security

    @Configuration
	@EnableWebSecurity
	@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
	@Import(SecurityProblemSupport.class)
	public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
		...
		private final SecurityProblemSupport problemSupport;
		...

		@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .addFilterBefore(corsFilter, UsernamePasswordAuthenticationFilter.class)
                .exceptionHandling()
                .authenticationEntryPoint(problemSupport)
                .accessDeniedHandler(problemSupport)
 				...

    	}
	}

关于[problem-spring-web](https://github.com/zalando/problem-spring-web/tree/master/problem-spring-web)

### 二、增加User 、Account

数据库关系表的设计可以由具体业务决定，此处只给出较为通用的设计。参考templates\security-with-account 中的domain，repository配置

### 三、Security配置

#### 新增security包

    security
        jwt
			JWTConfigurer.java
			JWTFilter.java
			TokenProvider.java
			
		DomainUserDetailsService.java
		SecurityUtils.java
#### 配置SecurityConfiguration
参考templates\security-with-account中的config

### 四、配置RestController

    @RestController
@RequestMapping("/api")
public class JWTController {

    private final AuthenticationService authenticationService;

    public JWTController(AuthenticationService authenticationService) {
        this.authenticationService = authenticationService;
    }

    @PostMapping("/authenticate")
    public ResponseEntity<Map<String,Object>> authorize(@RequestBody @Valid LoginVM loginVM) {
        String jwt = this.authenticationService.authenticate(loginVM);
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.add(JWTConfigurer.AUTHORIZATION_HEADER, "Bearer " + jwt);
        Map<String, Object> responseMapper = new HashMap<>();
        responseMapper.put("token", jwt);
        return new ResponseEntity<>(responseMapper, httpHeaders, HttpStatus.OK);
    }
}

##### 如何支持仅用户账户登录

##### 如何支持用户账户/手机号登录？
