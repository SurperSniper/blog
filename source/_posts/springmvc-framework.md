title: SpringMVC 
date: 2015-06-07 20:52:58
categories: Study Notes
tags: [springmvc]
---
# springmvc 介绍
springmvc是spring框架的一个模块，属于表现层框架，springmvc和spring无需通过中间整合层进行整合
![spring](http://7xjkgu.com1.z0.glb.clouddn.com/image/spring.png)
# 架构流程
1. 用户发送请求（request）至前端控制器DispatcherServlet
2. DispatcherServlet收到请求调用HandlerMapping处理器映射器
3. 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器（如果有则生成）并返回给DispatcherServlet
4. DispatcherServlet通过HandlerAdapter处理器适配器调用处理器
5. 执行处理器（Controller，也叫后端处理器）
6. Controller执行完成返回ModelAndView
7. HandlerAdapter将Controller执行结果ModelAndView返回给DispatcherServlet
8. DispatcherServlet将ModelAndView传给ViewResolver视图解析器
9. ViewResolver解析后返回具体View
10. DispatcherServlet将view进行视图渲染（将模型数据填充值视图中）
11. DispatcherServlet响应（response）用户  
# 组件说明
以下组件通常使用框架提供实现：

- DispatcherServlet：前端控制器	
						
 	用户请求到达前端控制器，相当于mvc模块中的c，DispatcherServlet是整个流程控制的中心，由它调用其他组件处理用户的请求，DispatcherServlet的存在降低了组件之间的耦合性
- HandlerMapping：处理器映射器
 
	HandlerMapping负责根据用户请求找到Handler，spingmvc提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等
- Handler：处理器
 
	Handler是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理
`由于Handler涉及到具体的用户业务请求，所以一般情况需要程序员根据业务需求开发Handler`
- HandlerAdapter:处理器适配器

	通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩张适配器可以对更多类型的处理器进行执行
- ViewResolver

	ViewResolver负责将处理结果生成View视图，ViewResolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。springmvc框架提供了很多的View视图类型，包括jstlView、freemarkerView、pdfView等
`一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面`
## DispatcherServlet 前端控制器
DispathcerServlet作为springmvc的中央调度器存在，DispatcherServlet创建时会默认从`DispatcherServlet.properties`文件加载springmvc所用的各个组件，如果在springmvc.xml中配置了组件则以springmvc.xml中配置的为准，DispatcherServlet的存在降低了springmvc各各组件之间的耦合度。
## HandlerMapping 处理器映射器
HandlerMapping 负责根据request请求找到对应的Handler处理器及Interceptor拦截器，将它们封装在HandlerExecutionChain 对象中给前端控制器返回。
## HandlerAdapter 处理器适配器
HandlerAdapter会根据适配器接口对后端控制器进行包装（适配），包装后即可对处理器进行执行，通过扩展处理器适配器可以执行多种类型的处理器，这里使用了适配器设计模式。
# 配置
## 组件扫描器
使用组件扫描器省去在spring容器配置每个controller类的繁琐。使用`<context:component-scan>`自动扫描标记`@controller`的控制器类，配置如下：

    <!-- 扫描controller注解,多个包中间使用半角逗号分隔 -->
    <context:component-scan base-package="cn.itcast.springmvc.controller.first"/>

## RequestMappingHandlerMapping
注解式处理器映射器，对类中标记`@ResquestMapping`的方法进行映射，根据ResquestMapping定义的url匹配ResquestMapping标记的方法，匹配成功返回HandlerMethod对象给前端控制器，HandlerMethod对象中封装url对应的方法Method。 

从spring3.1版本开始，废除了`DefaultAnnotationHandlerMapping`的使用，推荐使用`RequestMappingHandlerMapping`完成注解式处理器映射。

配置如下：

    <!--注解映射器 -->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>

## RequestMappingHandlerAdapter
注解式处理器适配器，对标记`@ResquestMapping`的方法进行适配。

从spring3.1版本开始，废除了`AnnotationMethodHandlerAdapter`的使用，推荐使用`RequestMappingHandlerAdapter`完成注解式处理器适配。

配置如下：

    <!--注解适配器 -->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>
## <mvc:annotation-driven>
springmvc使用`<mvc:annotation-driven>`自动加载`RequestMappingHandlerMapping`和`RequestMappingHandlerAdapter`，可用在springmvc.xml配置文件中使用`<mvc:annotation-driven>`替代注解处理器和适配器的配置。
# springmvc+mybatis整合
表现层：springmvc -> 业务层：Service接口 -> 持久层：mybatis -> mysql

>spring将各层进行整合
>>- 通过spring管理持久层的Mapper（Dao接口）
>>- 通过spring管理业务层Service，Service中可以调用Mapper接口
>>- spring进行事务控制
>- 通过spring管理表现层Handler，Handler中可以调用Service接口    

> Mapper、Service、Handler都是javabean

1. 整合dao层

	mybatis和spring整合，通过spring管理mapper接口。
	使用mapper的扫描器自动扫描mapper接口在spring中进行注册。
2. 整合service层

	通过spring管理 service接口。
	使用配置方式将service接口配置在spring配置文件中。
	实现事务控制。
3. 整合springmvc

	由于springmvc是spring的模块，不需要整合。
# spring参数绑定过程

----------

springmvc中，接收页面提交的数据是通过方法形参来接收，而不是在controller类定义成员变更接收！

注解适配器对`RequestMapping`标记的方法进行适配，对方法中的形参会进行参数绑定，早期springmvc采用PropertyEditor（属性编辑器，只能将字符串转换成java对象）进行参数绑定将request请求的参数绑定到方法形参上，3.X之后springmvc就开始使用Converter（任意类型的转换）进行参数绑定。
## 简单类型
当请求的参数名称和处理器形参名称一致时会将请求参数与形参进行绑定。
## 简单pojo
将pojo对象中的属性名于传递进来的属性名对应，如果传进来的参数名称和对象中的属性名称一致则将参数值设置在pojo对象中
### 自定义参数绑定实现日期类型绑定

对于controller形参中pojo对象，如果属性中有日期类型，需要自定义参数绑定。
将请求日期数据串传成日期类型，要转换的日期类型和pojo中日期属性的类型保持一致。
#### 自定义Converter

    public class CustomDateConverter implements Converter<String, Date> {
    	@Override
    	public Date convert(String source) {
    		try {
    			SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    			return simpleDateFormat.parse(source);
    		} catch (Exception e) {
    			e.printStackTrace();
    		}
    		return null;
    	}
    }
#### 配置方式

需要向处理器适配器中注入自定义的参数绑定组件

    <mvc:annotation-driven conversion-service="conversionService">
      </mvc:annotation-driven>
      <!-- conversionService -->
    	<bean id="conversionService"
    		class="org.springframework.format.support.FormattingConversionServiceFact  oryBean">
    	<!-- 转换器 -->
    	<property name="converters">
    		<list>
    			<bean class="cn.itcast.ssm.controller.converter.CustomDateConverter"/>
    		</list>
    	</property>
    </bean>

## 包装pojo
如果采用类似struts中对象.属性的方式命名，需要将pojo对象作为一个包装对象的属性，action中以该包装对象作为形参
# post中文乱码

----------

在web.xml中加入：

    <filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter
    </filter-class>
    <init-param>
    <param-name>encoding</param-name>
    <param-value>utf-8</param-value>
    </init-param>
    </filter>
    <filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
    </filter-mapping>

以上可以解决post请求乱码问题。
对于get请求中文参数出现乱码解决方法有两个：

修改tomcat配置文件添加编码与工程编码一致，如下：

``<Connector URIEncoding="utf-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>``
另外一种方法对参数进行重新编码：

`` String userName = new 
String(request.getParamter("userName").getBytes("ISO8859-1"),"utf-8")``
ISO8859-1是tomcat默认编码，需要将tomcat编码后的内容按utf-8编码
# springmvc和struts2的区别

----------
1. springmvc的入口是一个servlet即前端控制器，而struts2入口是一个filter过虑器
2. springmvc是基于方法开发(一个url对应一个方法)，请求参数传递到方法的形参；将url和controller方法映射，映射成功后springmvc生成一个Handler对象，对象中只包括了一个method，方法执行结束，形参数据销毁，可以设计为单例或多例(建议单例)

	struts2是基于类开发，传递参数是通过类的属性，只能设计为多例。
3. Struts采用值栈存储请求和响应的数据，通过OGNL存取数据， springmvc通过参数解析器是将request请求内容解析，并给方法形参赋值，将数据和视图封装成ModelAndView对象，最后又将ModelAndView中的模型数据通过reques域传输到页面。Jsp视图解析器默认使用jstl

