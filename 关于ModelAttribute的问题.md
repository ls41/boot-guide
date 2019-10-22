## 关于ModelAttribute的问题

### 一、缘由

在处理SpringMVC的GetMapping查询时，如果参数是多个时，需要单个单个的用@RequestParam处理，效率低下且难以维护，一直思考是否有更便捷的方式，在之前昆堪院的系统中通过自己实现HandlerMethodArgumentResolver，来达成参数自动转换成Bean的目的，但是自己实现的存在不少的问题，在面对复杂数据类型转换的时候比较无力，且不愿意公开自己的代码给现公司。百般思考后考虑优先找寻Spring的官方解决方案。

代码对比：

**丑陋的代码**

    @GetMapping
    private R query(@RequestParam String name1,
    @RequestParam String name2,
    @RequestParam String name3){
    CustomerQuery query = new CustomerQuery();
    query.setName(name1);
    query.setName(name2);
    query.setName(name2);
    
    return new R();
    }

**优雅的代码**

	@GetMapping
    private R list(CustomerQuery query){
        System.out.println(query.getName());
        return new R();
    }
	
	相当于

	@GetMapping
    private R list(@ModelAttribute CustomerQuery query){
        System.out.println(query.getName());
        return new R();
    }

### 二、刨根问底

经查询了StackOverflow不经意间找到如下的办法，
[传送门在此](https://stackoverflow.com/questions/41068956/java-spring-rest-api-handling-many-optional-parameters)
依据回答者的办法一试，果然没问题。但是为啥呢？

1、首先更改Spring的日志级别至TRACE

2、查询请求日志

找到关键信息`HandlerMethodArgumentResolverComposite`此对象用于匹配合适的参数处理器，它调用内部方法`getArgumentResolver`获取合适的参数处理器，通过断点可以查看优雅代码的示例使用的是`ServletModelAttributeMethodProcessor`参数处理器。