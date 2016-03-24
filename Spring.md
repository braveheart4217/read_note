##Spring
1. 体系结构
	1. 模块结构
	
		![](http://wiki.jikexueyuan.com/project/spring/images/arch1.png)
		1. 核心容器：核心，Bean，上下文与表达式语言
			* 核心模块包括框架的基本组成部分，包括 IoC与依赖注入
			* Bean 提供 BeanFactory
			* 上下文模块建立在核心与Bean之上，它是访问定义和配置的任何对象的媒介，ApplicationContext 接口是上下文模块的重点
			* 表达式语言模块在运行时提供了查询与操作一个对象图的强大表达式语言
		2. 数据访问/集成层包括 `JDBC，ORM，OXM，JMS `和事务处理模块：
			* JDBC 模块提供了删除冗余的 JDBC 相关编码的 JDBC 抽象层。
			* ORM 模块为流行的对象关系映射 API，包括 JPA，JDO，Hibernate 和 iBatis，提供了集成层。
			* OXM 模块提供了抽象层，它支持对 JAXB，Castor，XMLBeans，JiBX 和 XStream 的对象/XML 映射实现。
			* Java 消息服务 JMS 模块包含生产和消费的信息的功能。
			* 事务模块为实现特殊接口的类及所有的 POJO 支持编程式和声明式事务管理。
		3. Web 层由 Web，Web-MVC，Web-Socket 和 Web-Portlet 组成：
			* Web 模块提供了基本的面向 web 的集成功能，例如多个文件上传的功能和使用 servlet 监听器和面向 web 应用程序的上下文来初始化 IoC 容器。
			* Web-MVC 模块包含 Spring 的模型-视图-控制器（MVC），实现了 web 应用程序。
			* Web-Socket 模块为 WebSocket-based 提供了支持，而且在 web 应用程序中提供了客户端和服务器端之间通信的两种方式。
			* Web-Portlet 模块提供了在 portlet 环境中实现 MVC，并且反映了 Web-Servlet 模块的功能
		4. 其他一些模块
			* AOP 模块提供了面向方面的编程实现，允许你定义方法拦截器和切入点对代码进行干净地解耦，它实现了应该分离的功能。
			* Aspects 模块提供了与 AspectJ 的集成，这是一个功能强大且成熟的面向切面编程（AOP）框架。
			* Instrumentation 模块在一定的应用服务器中提供了类 instrumentation 的支持和类加载器的实现。
			* Messaging 模块为 STOMP 提供了支持作为在应用程序中 WebSocket 子协议的使用。它也支持一个注解编程模型，它是为了选路和处理来自 WebSocket 客户端的 STOMP 信息。
			* 测试模块支持对具有 JUnit 或 TestNG 框架的 Spring 组件的测试

2. IoC 容器
	1. Spring 容器是 Spring 框架的核心。容器将创建对象，把它们连接在一起，配置它们，并管理他们的整个生命周期从创建到销毁。Spring 容器使用依赖注入（DI）来管理组成一个应用程序的组件。这些对象被称为 Spring Beans
	2. 通过阅读配置元数据提供的指令，容器知道对哪些对象进行实例化，配置和组装。配置元数据可以通过 XML，Java 注释或 Java 代码来表示。(Spring工作视图如下)
	
		![Spring工作视图](http://wiki.jikexueyuan.com/project/spring/images/ioc1.jpg)
	3. 容器类型
		* Spring BeanFactory
			* 它是最简单的容器，给 DI 提供了基本的支持，它用 org.springframework.beans.factory.BeanFactory 接口来定义。BeanFactory 或者相关的接口，如 BeanFactoryAware，InitializingBean，DisposableBean，在 Spring 中仍然存在具有大量的与 Spring 整合的第三方框架的反向兼容性的目的
		* Spring ApplicationContext
			* 该容器添加了更多的企业特定的功能，例如从一个属性文件中解析文本信息的能力，发布应用程序事件给感兴趣的事件监听器的能力。该容器是由 org.springframework.context.ApplicationContext 接口定义
			* 常用实现
				* `FileSystemXmlApplicationContext` 
					* 该容器从 XML 文件中加载已被定义的 bean。在这里，你需要提供给构造器 XML 文件的完整路径
				* ClassPathXmlApplicationContext
					* 该容器从 XML 文件中加载已被定义的 bean。你不需要提供 XML 文件的完整路径，容器会从 CLASSPATH 中搜索 bean 配置文件
				* WebXmlApplicationContext
					* 该容器会在一个 web 应用程序的范围内加载在 XML 文件中已被定义的 bean

3. Bean定义
	1. bean 是一个被实例化，组装，并通过 Spring IoC 容器所管理的对象。这些 bean 是由用容器提供的配置元数据创建的。
	2. bean 定义包含称为 **配置元数据** 的信息，下述容器也需要知道配置元数据：
		1. 如何创建一个 bean
		2. bean 的生命周期的详细信息
		3. bean 的依赖关系
	3. Bean 的属性
	
		|属性|描述|
		|--|--|
		|class	|这个属性是强制性的，并且指定用来创建 bean 的 bean 类|
		|name | 指定唯一的 bean 标识符，也就是 BeanFactory 的 Key|
		|scope	|指定由特定的 bean 定义创建的对象的作用域|
		|constructor-arg	|它是用来注入依赖关系的
		|properties	|它是用来注入依赖关系的
		|autowiring mode|	它是用来注入依赖关系的
		|lazy-initialization mode	|延迟初始化的 bean,告诉 IoC 容器在它第一次被请求时，而不是在启动时去创建一个 bean 实例
		|initialization 方法 | 在 bean 的所有必需的属性被容器设置之后，调用回调方法
		|destruction 方法|destruction 方法

	4. Spring 配置元数据
		* Spring IoC容器获取元数据的三种方法
			* 基于 XML 的配置文件
			* 基于注解的配置
			* 基于Java的配置
	5. Bean 作用域
		1. Spring支持的作用域
			1. singleton
				* Spring IoC 维护该类的一个单一实例(单例)
			2. prototype
				* 每次请求都会实例化一个对象
			3. request
				* 该作用域将 bean 的定义限制为 HTTP 请求。只在 web-aware Spring ApplicationContext 的上下文中有效
			4. session
				* 该作用域将 bean 的定义限制为 HTTP 会话.(同上)
			5. 该作用域将 bean 的定义限制为 HTTP 会话
				* 该作用域将 bean 的定义限制为全局 HTTP 会话.(同上)

	6. Bean 生命周期
		1. 当一个 bean 被实例化时，它可能需要执行一些初始化使它转换成可用状态。同样，当 bean 不再需要，并且从容器中移除时，可能需要做一些清除工作
		2. `init-method` 属性指定一个方法，在实例化 bean 时，立即调用该方法，或者 `implements` `InitializingBean` 接口，然后实现 `afterPropertiesSet()`方法
		3. `destroy-method` 指定一个方法，只有从容器中移除 bean 之后，调用该方法，或者 `implement DisposableBean` 接口，然后实现 `destroy()` 方法
		4. 当作用域是 `prototype` 的时候无法执行 `destory` 方法
			* Spring不能对一个 `prototype bean` 的整个生命周期负责，容器在初始化、配置、装饰或者是装配完一个 `prototype` 实例后，将它交给客户端，随后就对该 `prototype` 实例不闻不问了。不管何种作用域，容器都会调用所有对象的初始化生命周期回调方法，而对 `prototype` 而言，任何配置好的析构生命周期回调方法都将不会被调用。 清除 `prototype` 作用域的对象并释放任何 `prototype bean` 所持有的昂贵资源，都是客户端代码的职责

	7. Bean 后置处理器
		1. `BeanPostProcessor` 接口定义回调方法，你可以实现该方法来提供自己的实例化逻辑，依赖解析逻辑
		2. 如果配置了多个 `BeanPostProcessor` ，可以通过设置 `BeanPostProcessor` 实现的 `Ordered` 接口提供的 `order` 属性来控制这些 `BeanPostProcessor` 接口的执行顺序
		3. `ApplicationContext` 会自动检测由 `BeanPostProcessor` 接口的实现定义的 bean，注册这些 bean 为后置处理器，然后通过在容器中创建 bean，在适当的时候调用它

	8. Bean 定义继承
		1. 通过 `parent` 属性来指明 父Bean
		2. 这里的继承与Java本身的继承类似，子Bean可以增加或者复写父Bean的 `property` 标签的属性值
		3. 可以通过指明 `abstract="true"` 来定义抽象父Bean，抽象父Bean不应该指明 `class` 属性

	9. 依赖注入
		1. 基于构造器的注入
			1. 示例
	
					public Foo(Bar bar, Baz baz);//foo 构造函数声明

					<beans>
	   					<bean id="foo" class="x.y.Foo">
	      					<constructor-arg ref="bar"/>  // 构造器的参数在Bean中定义的顺序就是把这些参数提供给适当构造函数的顺序
	      					<constructor-arg ref="baz"/>
	   					</bean>

	   					<bean id="bar" class="x.y.Bar"/>
	   					<bean id="baz" class="x.y.Baz"/>
					</beans>
			2. 可以通过 type 属性，指明传入参数的类型，或者通过index 显示指明参数索引
				
					public exampleBean(int year, String name); //foo 构造函数声明

					// 使用type
					<bean id="exampleBean" class="examples.ExampleBean">
				    	<constructor-arg type="int" value="2001"/>
				      	<constructor-arg type="java.lang.String" value="Zara"/>
				   	</bean>
					
					// 使用index
					<bean id="exampleBean" class="examples.ExampleBean">
				      	<constructor-arg index="0" value="2001"/>
				   		<constructor-arg index="1" value="Zara"/>
				   	</bean>

		2. 基于setter函数的注入
			1. 示例
			
					<!-- Definition for spellChecker bean -->
					<bean id="textEditor" class="com.tutorialspoint.TextEditor">
						<property name="spellChecker" ref="spellChecker"/>
					</bean>
				
					<!-- Definition for spellChecker bean -->
					<bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
					</bean>
			2. setter函数注入与构造器注入的区别
				* 唯一的区别就是在基于构造函数注入中，我们使用的是  〈constructor-arg〉 标签中的 〈bean〉 元素，而在基于设值函数的注入中，我们使用的是 〈property〉 标签中的 〈bean〉 元素
				* 注意：如果你要把一个引用传递给一个对象，那么你需要使用 标签的 **`ref`** 属性，而如果你要直接传递一个值，那么你应该使用 **`value`** 属性

			3. 使用 `p-namespace` 实现 XML 的配置
				1. 标准形式
					
						<bean id="john-classic" class="com.example.Person">
					       <property name="name" value="John Doe"/>
					       <property name="spouse" ref="jane"/>
					    </bean>
					
					    <bean name="jane" class="com.example.Person">
					        <property name="name" value="John Doe"/>
					    </bean>
				2. `p-namespece` 形式
				
						<bean id="john-classic" class="com.example.Person"
					       p:name="John Doe"
					       p:spouse-ref="jane"/>
					    </bean>
					
					    <bean name="jane" class="com.example.Person"
					       p:name="John Doe"/>
					    </bean>

	10. 注入内部 `Beans`
		1. 示例
			
				<bean id="outerBean" class="...">
			       <property name="target">
			          <bean id="innerBean" class="..."/>
			       </property>
			    </bean>
	11. 注入集合
		1. Spring 提供了四种类型的集合的配置元素
			1. `<list>` : 链表
			2. `<set>` : 集合
			3. `<map>` : key-value 是任意类型
			4. `<props>` : key-value 都是 字符串类型
		2. 示例
	
				<property name="addressList">
		        	<list>
		            	<value>INDIA</value>
		            	<value>Pakistan</value>
		            	<value>USA</value>
		            	<value>USA</value>
		         	</list>
		      	</property>
		3. 注入Bean引用
			1. 示例
			
					<property name="addressList">
			         	<list>
			            	<ref bean="address1"/>  // 引用Bean的ID为address1的Bean
			            	<ref bean="address2"/>
			            	<value>Pakistan</value>
			         	</list>
			      	</property>

					// 注意与上面不同的地方`(value-ref)`
					<property name="addressMap">
			         	<map>
			            	<entry key="one" value="INDIA"/>
			            	<entry key ="two" value-ref="address1"/>
			            	<entry key ="three" value-ref="address2"/>
			         	</map>
			      	</property>
		4. 注入`NULL`与空字符串
			1. 注入 NULL

					<bean id="..." class="exampleBean">
   						<property name="email"><null/></property>
						<property name="email"><null/></property>
					</bean>
			2. 注入空字符串

					<bean id="..." class="exampleBean">
   						<property name="email" value=""/>
						<property name="email" value=""/>
					</bean>
	1. Beans 自动装配
		1. Spring可以通过一些方式自动注入值
			1. byName
				* 通过属性名自动装配，`Spring` 会自动寻找 `Bean id` 值与这个 Bean 中属性名相同的 `Bean`，然后进行连接(就是会自动注入)
			2. byType
				* 属性数据类型自动装配，如果它的类型匹配配置文件中的一个确切的 bean 名称，它将尝试匹配和连接属性的类型
			3. constructor
			4. autodetect
				* Spring首先尝试通过 constructor 使用自动装配来连接，如果它不执行，Spring 尝试通过 byType 来自动装配

		2. 示例
			1. byName
				1. 如果存在一个和当前属性名字一致的 Bean，则使用该 Bean 进行注入。如果名称匹配但是类型不匹配，则抛出异常。如果没有匹配的类型，则什么也不做

						// 在注入的时候， 注入器发现 TextEditor 有两个属性值需要输入，但是Bean中只描述
						// name 属性的注入方式，然后注入器又发现，Bean 指定了 autowire="byName"
						// 于是注入器开始寻找是否含有 Bean id 是属性名的Beans，如果有就会在请求Bean的时
						// 候自动注入(个人理解) 
						<bean id="textEditor" class="com.tutorialspoint.TextEditor" 
						    autowire="byName">
							
							// 如果不使用 autowire ，应该要加上这句
							// <property name="spellChecker" ref="spellChecker" />
	
						   <property name="name" value="Generic Text Editor" />
						</bean>
						
						<bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
						</bean>
				
			2. byType
				1.  如果存在一个和当前属性类型一致的 Bean ( 相同类型或者子类型 )，则使用该 Bean 进行注入。byType 能够识别工厂方法，即能够识别 factory-method 的返回类型。如果存在多个类型一致的 Bean，则抛出异常。如果没有匹配的类型，则什么也不做
					
						// 过程与 byType 相似，只不过最后注入器寻找的是 
						// 以被注入对象的 类型名 为id 的Bean
					    <bean id="textEditor" class="com.tutorialspoint.TextEditor" 
					        autowire="byType">
					        <property name="name" value="Generic Text Editor" />
					    </bean>
						
						// 注意 被注入的 Bean 的 id 值，应该是对应 Class 的类型
					    <bean id="SpellChecker" class="com.tutorialspoint.SpellChecker">
					    </bean>
			3. 构造函数自动注入
				* 此种方式与 `byType` 方式很像，可以理解为构造器注入的 `byType`，就是构造器会根据参数类型自动寻找以对应参数类型为 id 的 Bean，然后注入
				* 与 byType 类似，只不过它是针对构造函数注入而言的。如果当前没有与构造函数的参数类型匹配的 Bean，则抛出异常。使用该种装配模式时，优先匹配参数最多的构造函数
				* 示例
				
						// 根据构造函数参数类型作为 id 寻找 Bean
						<bean id="textEditor" class="com.tutorialspoint.TextEditor" 
					       autowire="constructor">

							// 正常情况下应该会有这句话
							// <constructor-arg  ref="spellChecker" />

					       <constructor-arg value="Generic Text Editor"/>
					    </bean>

					    <bean id="SpellChecker" class="com.tutorialspoint.SpellChecker">
					    </bean>

			4. autodetect
				* 根据 Bean 的自省机制决定采用 byType 还是 constructor 进行自动装配。如果 Bean 提供了默认的构造函数，则采用 byType；否则采用 constructor 进行自动装配

	1. 基于注解的配置
		1. 打开注解 注入
			
				<context:annotation-config/>
		2. 几个重要的注解
			1. `@Required`
				* `@Required` 注解应用于 bean 属性的 setter 方法
				* 它表明受影响的 `bean` 属性在配置时必须放在 XML 配置文件中，否则容器就会抛出一个 `BeanInitializationException` 异常
			2. `@Autowired`
				* `@Autowired` 注解可以应用到 bean 属性的 `setter` 方法，构造函数和属性
				* 当 `@Autowired` 标注于 setter方法，属性值，构造函数时，注入器会尝试自动执行 `byType` (默认按类型匹配) 自动连接(经过测试好像都可以，`byName` 或者 `byType` )
			3. `@Qualifier`
				* 通过指定确切的将被注入的 bean，就是当一个接口有多个实现类的时候，可以通过 `@qulifier` 指定你此时需要注入的类
				
						@Autowired
					    @Qualifier("student1") // 指定实现 Bean 的 Id
					    private Student student;


			4. 	`JSR-250 Annotations`
				* Spring 支持 JSR-250 的基础的注解，其中包括了 `@Resource`，`@PostConstruct` 和 `@PreDestroy` 注解
				* `@PostConsturct` 与 `@PreDestroy` 注解 就是用以代替 `Bean` 里面的 `init-method` 与 `destroy-method` 属性
				* @Resource
					* 你可以在字段中或者 `setter` 方法中使用 `@Resource` 注释
					* `@Resource` 注释使用一个 `"name"` 属性，该属性以一个 `bean` 名称的形式被注入。你可以说，它遵循 `byName` 自动连接语义
					* 如果没有明确地指定一个 `"name"`，默认名称源于字段名或者 `setter` 方法
						* 在字段的情况下，它使用的是字段名；在一个 `setter` 方法情况下，它使用的是 `bean` 属性名称

		3. 注意
			1. 如果要开启注解支持，需要在 `Beans.xml` 文件中增加如下代码

					<context:annotation-config/>
			2. 但是有时候即使你加了以上代码，Beans.xml文件还是会报错，例如：
				
					The prefix "context" for element "context:annotation-config" is not bound.
					//这个错误的意思是 你没有没有申明 context 标签，然后就使用了
			3. 然后你必须 在 Beans 标签里添加属性

					xmlns:context="http://www.springframework.org/schema/context"
			4. 有时候即使你上面都做了，还是会报错：

					Context namespace element 'annotation-config' and its parser class [org.springframework.context.annotation.AnnotationConfigBeanDefinitionParser] are only available on JDK 1.5 and higher
			5. 报这个错误的原因一般你使用的 `Spring` 版本太低，`Java` 版本太高，导致 `Spring` 误将你的 `Java` 版本识别成了 Java1.4，然后就报错了，解决办法：
				1. 将你的 `Spring` 版本升级
				2. [点我传送](http://my.oschina.net/aruan/blog/210284) 

	1. 基于 Java 的配置
		1. 基于 Java 的配置选项，可以使你在不用配置 XML 的情况下编写大多数的 Spring 
			1. `@Configuration`
				* 被 `@Configuration` 的注解类 表示这个类可以使用 `Spring IoC` 容器作为 bean 定义的来源
			2. @Bean
				* `@Bean` 注解告诉 `Spring`，一个带有 `@Bean` 的注解方法将返回一个对象，该对象应该被注册为在 `Spring` 应用程序上下文中的 `bean`

						package com;
						@Configuration
						public class HelloWorldConfig {
   						@Bean 
						public HelloWorld helloWorld(){
						    return new HelloWorld();
						}

						// 相当于
			
						<beans>
						   <bean id="helloWorld" class="com.HelloWorld" />
						</beans>
				* 带有 `@Bean` 注解的方法名称作为 `bean` 的 `ID`，它创建并返回实际的 `bean`。你的配置类可以声明多个 `@Bean`。一旦定义了配置类，你就可以使用 `AnnotationConfigApplicationContext` 来加载并把他们提供给 `Spring` 容器
							
						// 将 HelloConfig 类里面配置的 Bean 添加到 Spring 容器
						ApplicationContext ctx = 
   							new AnnotationConfigApplicationContext(HelloConfig.class); 
						// 获取 Bean
   						HelloWorld helloWorld = ctx.getBean(HelloWorld.class);
			3. 注入 Bean 的依赖性
			
					// 通过调用被 @Bean 标识的函数来注入 Bean 依赖
					@Bean 
   					public TextEditor textEditor(){
      					return new TextEditor( spellChecker() );
   					}
   					@Bean 
   					public SpellChecker spellChecker(){
      					return new SpellChecker( );
   					}

			4. `@Import`
				1. `@import` 注解允许从另一个配置类中加载 `@Bean` 定义，[点我详情](http://wiki.jikexueyuan.com/project/spring/java-based-configuration.html)
			5. 生命周期回调
				* `@Bean` 注解支持指定任意的初始化和销毁的回调方法，就像在 `bean` 元素中 `Spring` 的 `XML` 的初始化方法和销毁方法的属性
				
						public class Foo {
						   public void init() {
						      // initialization logic
						   }
						   public void cleanup() {
						      // destruction logic
						   }
						}
						
						@Configuration
						public class AppConfig {
						   @Bean(initMethod = "init", destroyMethod = "cleanup" )
						   public Foo foo() {
						      return new Foo();
						   }
						}
			6. 指定 `Bean` 的范围可以通过 `@Scope` 注解，默认是单例

					@Bean
				    @Scope("prototype")
				    public Foo foo() {
				       return new Foo();
				    }

	1. `Spring` 中的事件处理
		1. `Spring` 通过 `ApplicationEvent` 类和 `ApplicationListener` 接口来提供在 `ApplicationContext` 中处理事件。如果一个 `bean` 实现 `ApplicationListener`，那么每次 `ApplicationEvent` 被发布到 `ApplicationContext` 上时，那个 `bean` 会被通知
		2. Spring 标准事件
			1. `ContextRefreshedEvent`
				* `ApplicationContext` 被初始化或刷新时，该事件被发布
			2. `ContextStartedEvent`
				* 当使用 `ConfigurableApplicationContext` 接口中的 `start()` 方法启动 `ApplicationContext` 时，该事件被发布
			3. `ContextStoppedEvent`
				* 当使用 `ConfigurableApplicationContext` 接口中的 `stop()` 方法停止 `ApplicationContext` 时，发布这个事件
			4. `ContextClosedEvent`
				* 当使用 `ConfigurableApplicationContext` 接口中的 `close()` 方法关闭 `ApplicationContext` 时，该事件被发布
				* 一个 已关闭 的上下文到达 **生命周期末端**；它不能被刷新或重启
			5. `RequestHandledEvent`
				* 这是一个 `web-specific` 事件，告诉所有 `bean` `HTTP` 请求已经被服务
		3. 监听上下文事件
			1. 为了监听上下文事件，一个 `bean` 应该实现只有一个方法 `onApplicationEvent()` 的 `ApplicationListener` 接口
			2. TODO 为什么 ContextStop 事件没有被发送? 

		4. Spring 自定义事件
			* 具体内容见工程 `testSpring.customize.event`

	1. `Spring AOP`
		1. AOP 术语
			1. 通知（Advice）
				*  就是你想要的功能，也就是上面说的 安全，事物，日志等
			2. 连接点（JoinPoint）
				* 就是spring允许你使用通知的地方，那可真就多了，基本每个方法的前，后（两者都有也行），或抛出异常时都可以是连接点，spring只支持方法连接点.其他如aspectJ还可以让你在构造器或属性注入时都行，不过那不是咱关注的，只要记住，和方法有关的前前后后（抛出异常），都是连接点
			3. 切入点（PointCut）
				* 上面说的连接点的基础上，来定义切入点，你的一个类里，有15个方法，那就有几十个连接点了对把，但是你并不想在所有方法附近都使用通知（使用叫织入，以后再说），你只想让其中的几个，在调用这几个方法之前，之后或者抛出异常时干点什么，那么就用 切入点 来定义这几个方法，让 切入点 来筛选连接点，选中那几个你想要的方法
			4. 切面（Aspect）
				* 切面是通知和切入点的结合。现在发现了吧，没连接点什么事情，连接点就是为了让你好理解切入点，搞出来的，明白这个概念就行了
				* 通知说明了干什么和什么时候干（什么时候通过方法名中的 `before,after,around` 等就能知道），而切入点说明了在哪干（指定到底是哪个方法），这就是一个完整的切面定义
			5. 引入（introduction）
				* 允许我们向现有的类添加新方法属性。这不就是把切面（也就是新方法属性：通知定义的）用到目标类中吗
			6. 目标（target）
				* 引入中所提到的目标类，也就是要被通知的对象，也就是真正的业务逻辑，他可以在毫不知情的情况下，被咱们 织入切面。而自己专注于业务本身的逻辑
			7. 代理（proxy）
				* 怎么实现整套aop机制的，都是通过代理
			8. 织入（weaving）
				* 把切面应用到目标对象来创建新的代理对象的过程。有3种方式，spring采用的是运行时，为什么是运行时，后面解释
			9. 关键点
				* 切入点定义了哪些连接点会得到通知
		2. 对 AOP 的理解
			* spring用代理类包裹切面，把他们织入到Spring管理的bean中。也就是说代理类伪装成目标类，它会截取对目标类中方法的调用，让调用者对目标类的调用都先变成调用伪装类，伪装类中就先执行了切面，再把调用转发给真正的目标bean，那么如何才能搞出这个伪装类才不会被调用者发现（通过 JVM 的检查，Java是强类型检查，哪都要检查类型）
				1. 实现和目标类相同的接口
					* 实现了相同的接口，反正上层都是接口级别的调用，这样我就伪装成了和目标类一样的类（实现了同一接口，咱是兄弟了），也就逃过了类型检查，到java运行期的时候，利用多态的后期绑定（所以spring采用运行时），伪装类（代理类）就变成了接口的真正实现，而他里面包裹了真实的那个目标类，最后实现具体功能的还是目标类，只不过伪装类在之前干了点事情（写日志，安全检查，事物等）
					* 这就好比，一个人让你办件事，每次这个时候，你弟弟就会先出来，当然他分不出来了，以为是你，你这个弟弟虽然办不了这事，但是他知道你能办，所以就答应下来了，并且收了点礼物（写日志），收完礼物了，给把事给人家办了啊，所以你弟弟又找你这个哥哥来了，最后把这是办了的还是你自己。但是你自己并不知道你弟弟已经收礼物了，你只是专心把这件事情做好
				2. 如果本身这个类没有实现一个接口，那就只能创建一个目标类的子类
					* 生成子类调用，这次用子类来做为伪装类，当然这样也能逃过JVM的强类型检查，我继承的吗，当然查不出来了，子类重写了目标类的所有方法，当然在这些重写的方法中，不仅实现了目标类的功能（调用 `Super.function`），还在这些功能之前，实现了一些其他的（写日志，安全检查，事物等）
					* 这次的对比就是，儿子先从爸爸那把本事都学会了，所有人都找儿子办事情，但是儿子每次办和爸爸同样的事之前，都要收点小礼物（写日志），然后才去办真正的事。当然爸爸是不知道儿子这么干的了。这里就有件事情要说，某些本事是爸爸独有的(final的)，儿子学不了，学不了就办不了这件事，办不了这个事情，自然就不能收人家礼了
				3. 前一种兄弟模式，`spring` 会使用JDK的 `java.lang.reflect.Proxy` 类，它允许`Spring`动态生成一个新类来实现必要的接口，织入通知，并且把对这些接口的任何调用都转发到目标类
				4. 后一种父子模式，`spring`使用 `CGLIB` 库生成目标类的一个子类，在创建这个子类的时候，spring织入通知，并且把对这个子类的调用委托到目标类

参考资料：

1. [详解 Spring 3.0 基于 Annotation 的依赖注入实现](http://www.ibm.com/developerworks/cn/opensource/os-cn-spring-iocannt/) 
2. [Spring 参考手册](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-autowired-annotation)