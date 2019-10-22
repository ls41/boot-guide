# Swagger 配置

Swagger是一个自动生成API文档的工具，我们几乎在每个项目中都会使用到，它的配置简单且方便，为方便使用，我编写简单的文档描述它的配置方式。

### 配置Maven

**1、配置properties**

    <properties>
		...
		<swagger.version>2.9.2</swagger.version>
		<swagger.scope>compile</swagger.scope>
		...
    </properties>

配置properties 是为区分dev, prod 不同环境时，是否将swagger作为依赖jar与项目一起打包。

**2、配置dependency**

    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>${swagger.version}</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>swagger.version</version>
    </dependency>

**3、配置profiles**

    <profiles>
        <!--开发环境-->
        <profile>
            <id>dev</id>
            <properties>
                <profiles.active>dev</profiles.active>
                <maven.test.skip>true</maven.test.skip>
                <swagger.scope>compile</swagger.scope>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <profile>
            <!--生产环境-->
            <id>prod</id>
            <properties>
                <profiles.active>prod</profiles.active>
                <maven.test.skip>true</maven.test.skip>
                <swagger.scope>provided</swagger.scope>
            </properties>
        </profile>
    </profiles>
### 配置 application.yml

    spring:
      profiles:
    	active: @profiles.active@

@profiles.active@ 读取的是 profile->properties->profiles.active中的值，可以通过运行 `mvn package -Pprod` 设置active参数

### 配置Swagger

**1、 配置SwaggerConfiguaration**

    @Configuration
	@EnableSwagger2
	public class SwaggerConfiguration {
	    @Bean
	    @Profile("dev")
	    public Docket ProductApiDev() {
	        return new Docket(DocumentationType.SWAGGER_2)
	                .genericModelSubstitutes(DeferredResult.class)
	                .useDefaultResponseMessages(false)
	                .forCodeGeneration(false)
	                .pathMapping("/")
	                .select()
	                .build()
	                .apiInfo(productApiInfo());
	    }

	    @Bean
	    @Profile("prod")
	    public Docket ProductApiProd() {
	        return new Docket(DocumentationType.SWAGGER_2).enable(false);
	    }

	    private ApiInfo productApiInfo() {
	        return new ApiInfo("xxx文档名称",
	                "xxxx说明",
	                "1.0.0",
	                "API TERMS URL",
	                new Contact("刘斌", "", "xxxx@qq.com"),
	                "",
	                "",
	                new ArrayList<>());
	    }
    }

**2、配置WebConfig**

如`WebConfig`继承自 `WebMvcConfigurer`，则需要配置`addResourceHandlers`

	/**
     * 修复因继承的adapter导致的swagger的错误
     * @param registry
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
        registry.addResourceHandler("/**")
                .addResourceLocations("classpath:/static/");
    }