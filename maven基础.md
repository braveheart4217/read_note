Mavne使用

##Super Pom
所有的POM都继承与一个 Super POM文件，此文件中存储maven的一些默认设置，其帮助开发者在pom.xml中尽量少的做配置

可以使用一下命令来查看SuperPom内容(前提是pom.xml文件内容为默认类容)

	mvn help:effiective-pom

##构建生命周期
构建生命周期是一组阶段的序列（sequendce of phases）,每个阶段定义了目标被执行的顺序，这里的阶段是生命周期的一部分。

典型的Maven构建生命周期如下：
###################
| 阶段               |         处理      |    描述  
|-------------------------------------------------------------
| prepare-resources  |       资源拷贝    | 本阶段可以自定义需要拷贝的资源
| compile | 编译| 本阶段完成源代码的编译
|package | 打包 | 根据完pom.xml中的描述的打包配置创建JAR/WAR文件
|install| 安装| 在本地或者远程仓库中安装工程包


maven有以下几个标准的生命周期，当maven开始构建工程时，会按照所定义的阶段序列的顺序执行每个阶段注册的目标。

1. clean
2. default(or build)
3. site

以上的每一个构建生命周期通过一些 构建阶段(build phase) 来定义，其中每一个构建阶段(build phase)代表了生命周期中的一个阶段(Stage)


For example, the default lifecycle(build生命周期) has the following build phases (这里并不包含所有的phase，只是举例):

1. validate - validate the project is correct and all necessary information is available

2. compile - compile the source code of the project
3. test - test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed

4. package - take the compiled code and package it in its distributable format, such as a JAR.

5. integration-test - process and deploy the package if necessary into an environment where integration tests can be run

6. verify - run any checks to verify the package is valid and meets quality criteria

7. install - install the package into the local repository, for use as a dependency in other projects locally

8. deploy - done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.

**执行一个phase将会执行这个lifecycle前面的所有phase。**

下面这个例子执行了clean lifecycle和default lifecycle到install为止的所有phase。 

	mvn clean install

## 目标与构建阶段

1. A build phase is made up of goals
2. A goal represents a specific task (finer than a build phase) which contributes to the building and managing of a project

以上可知，执行phase实际上是执行goal。如果一个phase没有绑定goal，那么这个phase就不会被执行，
一些phase默认已经绑定了一下goal，对于default lifecircle来说，这些被绑定的goal并不完全相同，而是和packaging value相关。
	
	<packaging>jar</packaging> //当然对于不同的包类型，执行的过程不一样，执行的goal当然不一样

一个goal 是独立的，其可以被绑定到多个phase中去，也可以不绑定。如果一个goal没有被绑定到任何一个lifecircle，那他任然可以被直接调用。

因此，phase其实就是goal的容器，phase被执行的时候就是执行被绑定到该phase中的goal，goal与goal之间是独立的

	mvn clean dependency:copy-dependencies package

	clean 是 phase。
	dependency:copy-dependencies 是 plugin-in dependency 的 goal copy-dependencies。(dependency 是 plugin-in，copy-dependencies 是 goal)

	package也是一个phase。

maven会顺序执行这3个对象中包含的所有goal。

## Plugin
另一种将goal绑定到phase的方法就是在project中使用plugin，即在POM中配置

> Plugins are artifacts that provide goals to Maven. Furthermore, a plugin may have one or more goals wherein(其中) each goal represents a capability of that plugin

goal其实是存在于Maven的Repository中的plugin提供的一个个小的功能程序，它是Maven的lifecircle以及phase的基本组成元素，同时，我们也可以通过各种各样的goa加入到Maven的phase中，从而根据自己的实际需求实现各种功能

下面是一个更general的例子，该POM中展示了，如何将display-maven-plugin中的goal time绑定到process-test-resources这个phase中。

	 <plugin>
	   <groupId>com.mycompany.example</groupId>
	   <artifactId>display-maven-plugin</artifactId>
	   <version>1.0</version>
	   <executions>
	     <execution>
	       <phase>process-test-resources</phase>
	       <goals>
	         <goal>time</goal>
	       </goals>
	     </execution>
	   </executions>
	 </plugin>

##重要概念

1. 当一个阶段通过 Maven 命令调用时，例如 mvn compile，只有该阶段之前以及包括该阶段在内的所有阶段会被
执行。
2. 不同的 maven 目标将根据打包的类型（JAR / WAR / EAR），被绑定到不同的 Maven 生命周期阶段



当我们执行Maven命令时，Maven按照如下的顺序查找依赖库：本地仓库>中央仓库>远程仓库

##Maven插件
1. 插件的执行语法

	mvn [plugin-name]:[goal-name]

2. 插件类型 

|     类型     | 描述   
|---------------|-----------------|
|  Build plugins  |   在构建时执行，并在pom.xml的元素中配置| 
|  Reporting plugins | 在网站生成的过程中执行，并在pom.xml的元素中配置

3. 常用插件[^1]

|插件 | 描述|
|-------|------
|clean |构建之后清理目标文件。删除目标目录。
|compiler |编译 Java 源文件。
|surefile |运行 JUnit 单元测试。创建测试报告。
|jar |从当前工程中构建 JAR 文件。
|war |从当前工程中构建 WAR 文件。
|javadoc |为工程生成 Javadoc。
|antrun |从构建过程的任意一个阶段中运行一个 ant 任务的集合。

关键概念

:	插件是在pom.xml文件中的plugins元素定义的
:	每个插件可以有(绑定)多个目标
:	你可以定义阶段，插件会使用它的phase元素来处理任务
:	你可以通过绑定到插件的目标的方式来配置要执行的任务


当一个jar的外部依赖项无法存放在中央仓库或者远程仓库时可以:

1. 在src文件夹下面新建 lib 文件夹
2. 复制任何jar文件到 lib 文件夹下
3. 修改pom.xml文件，如下(以ldapjak为例)

		<dependency>
			<groupId>ldapjdk</groupId>
			<artifactId>ldapjdk</artifactId>
			<scope>system</scope>  // 作用域
			<version>1.0</version>
			<systemPath>${basedir}\src\lib\ldapjdk.jar</systemPath>  //相对路径
		</dependency>

##工程模板
使用如下命令快速创建一个java项目

	mvn archetype:generate

##快照
快照表示当前开发的一个副本，与常规版本不同，Maven为每一次构建从远程仓库中检出一份新的快照版本

**快照 VS 版本**

对于版本，maven一旦下载了指定的版本(eg:1.0)，它将不会尝试从仓库里面下载一个新的同样版本(1.0版本)文件，如果想要下载新的代码，必须升级版本。

对于快照,Maven将自动获取新的快照(data-service:1.0-SNAPSHPT)

##构建自动化

情景：app-web-ui 与 app-desktop-ui 项目依赖，bus-core-api

需求:app-web-ui 与 app-desktop-ui 团队需要保证当bus-core-api工程有变化的时候他们自己相应的工程可以随时被构建

我们有 2 种方式：

1. 在 bus-core-api 的 pom 文件里添加一个编译目标来提醒 app-web-ui 工程和 app-desktop-ui 工程启
动创建。
2. 使用一个持续集成（CI）的服务器，比如 Hudson，来实现自动化创建

###使用Maven方式

如下更新bus-core-api项目的pom.xml文件

	<build>
		<plugins>
			<plugin>
				<artifactId>maven-invoker-plugin</artifactId>
				<version>1.6</version>
				<configuration>
				<debug>true</debug>
					<pomIncludes>
						<pomInclude>app-web-ui/pom.xml</pomInclude>
						<pomInclude>app-desktop-ui/pom.xml</pomInclude>
					</pomIncludes>
				</configuration>
				<executions>
					<execution>
						<id>build</id>
						<goals>
							<goal>run</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	<build>

然后进入bus-core-api项目目录下，执行命令：

	mvn clean package -U
当`bus-core-api`构建完成之后，会自动构建app-web-api与app-desktop-api项目


##依赖管理
maven提供的控制可传递依赖的程度的功能：

| 功能  | 功能描述 |
|------|:-----------------------------------------------------|
|依赖调节|决定当多个手动创建的版本同时出现时，哪个依赖版本将会被使用。如果两个依赖版本在依赖树里的深度是一样的时候，第一个被声明的依赖将会被使用
|依赖管理|直接的指定手动创建的某个版本被使用。例如当一个工程 C 在自己的以来管理模块包含工程 B，即 B 依赖于 A， 那么 A 即可指定在 B 被引用时所使用的版本。
|依赖范围 | 包含在构建过程每个阶段的依赖。
|依赖排除 | 任何可传递的依赖都可以通过 "exclusion" 元素被排除在外。举例说明，A 依赖 B， B 依赖 C，因此 A可以记 C 为 “被排除的”。
|依赖可选|任何可传递的依赖可以被标记为可选的，通过使用 "optional" 元素。例如：A 依赖 B， B 依赖 C。因此，B 可标记 C 为可选的， 这样 A 就可以不再使用 C。

注意如下几点：

1. 公共的依赖可以使用pom父的概念被统一放在一起
2. maven通过使用可传递的依赖机制来实现项目间的依赖


## Syntax Of POM

依赖范围：
	
* maven在编译项目主代码的时候需要使用一套 classpath ，在编译和执行测试的时候也需要一套 classpath ，在最后实际运行maven项目的时候classpath又不一样，依赖范围(scope)就是来控制依赖与这三种classpath（编译 classpath ，测试classpath，运行classpath）的关系，具体的scope见下面

Dependency Scope

1. Compile
	> `for compilation, testing, and execution,and it's the default scope`
2. test
	> `only using for testing`
3. runtime
	> `for execute and test , not required for compilation`
4. provided
	> `the dependence is provided by JDK or a container,it will not package into the WAR`
5. system
	> `this is used to include external jars which can not be in center or remote repository`
#########################################################

1. compile
	> 默认的scope，表示 dependency 都可以在生命周期中使用。而且，这些dependencies 会传递到依赖的项目中。适用于所有阶段，会随着项目一起发布
2. provided
	> 跟compile相似，但是表明了dependency 由JDK或者容器提供，例如Servlet AP和一些Java EE APIs。这个scope只能作用在编译和测试时，同时没有传递性.
3. runtime
	> 表示dependency不作用在编译时，但会作用在运行和测试时，如JDBC驱动，适用运行和测试阶段。
4. test
	> 表示dependency作用在测试时，不作用在运行时。 只在测试时使用，用于编译和运行测试代码。不会随项目发布。
5. system
	> 跟provided 相似，但是在系统中要以外部JAR包的形式提供，maven不会在repository查找它(必须指定 `systemPath` 属性)。
6. import
	> 导入依赖范围，该依赖范围不会对三种 `classpath` 产生实际影响
 
##################################

[Maven中的基本概念](http://blog.csdn.net/onlyqi/article/details/6801318)

[maven基本命令，配置和扩展](http://blog.greekw.com/2016/01/04/maven%E5%9F%BA%E6%9C%AC%E5%91%BD%E4%BB%A4%EF%BC%8C%E9%85%8D%E7%BD%AE%E5%92%8C%E6%89%A9%E5%B1%95/)