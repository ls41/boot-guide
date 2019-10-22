## 基于Spring Data 配置AuditorAware

Spring Data提供支持审计功能：即由谁在什么时候创建或修改实体。Spring Data提供了在实体类的属性上增加`@CreatedBy`，`@LastModifiedBy`，`@CreatedDate`，`@LastModifiedDate`注解，并配置相应的配置项，即可实现审计功能，有系统自动记录`createdBy` `CreatedDate` `lastModifiedBy` `lastModifiedDate`四个属性的值，下面为具体的配置项。

### 实体类

    @EntityListeners(AuditingEntityListener.class)
    public abstract class AbstractAuditingEntity implements Serializable {
    
	    private static final long serialVersionUID = 1L;
	    
	    @CreatedBy
	    @Column(name = "created_by", nullable = false, length = 50, updatable = false)
	    private String createdBy;
	    
	    @CreatedDate
	    @Column(name = "created_date", nullable = false, updatable = false)
	    private Instant createdDate = Instant.now();
	    
	    @LastModifiedBy
	    @Column(name = "last_modified_by", length = 50)
	    private String lastModifiedBy;
	    
	    @LastModifiedDate
	    @Column(name = "last_modified_date")
	    private Instant lastModifiedDate = Instant.now();
		
		...
### AuditorAware 实现

新建`AuditingEntityConfiguration`类，并使用如下配置

    @Component
    public class AuditingEntityConfiguration implements AuditorAware<String> {
    
    @Override
    public Optional<String> getCurrentAuditor() {
    		return SecurityUtils.getCurrentUserLogin();
    	}
    }

### 启用审计功能

配置 `EnableJpaAuditing` 注解

    @SpringBootApplication
	...
    @EnableJpaAuditing
	...
    public class Application {
    
    public static void main(String[] args) {
    		SpringApplication.run(Application.class, args);
    	}
    }

> 注：在异步方法中如何获取用户信息
> 由于在异步方法中使用repository保存对象，获取不到用户用户信息，需增加如下配置项，即可在Authentication获取用户的信息
> 
>   
    @Bean
    public MethodInvokingFactoryBean methodInvokingFactoryBean() {
	    MethodInvokingFactoryBean methodInvokingFactoryBean = new MethodInvokingFactoryBean();
	    methodInvokingFactoryBean.setTargetClass(SecurityContextHolder.class);
	    methodInvokingFactoryBean.setTargetMethod("setStrategyName");
	    methodInvokingFactoryBean.setArguments(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL);
	    return methodInvokingFactoryBean;
    }