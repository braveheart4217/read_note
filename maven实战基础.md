##基本概念
1. 依赖与坐标
	1. 传递性依赖与依赖范围
		1. 依赖范围不仅可以控制依赖与三种classpath的关系，还会回传递性依赖产生影响
		2. 第一直接依赖与第二直接依赖的依赖范围决定了传递性依赖的范围（A依赖B，B依赖C，A对于B是第一直接依赖，B对C是第二直接依赖）
		3. 关系表如下(最左边的一列表示第一直接依赖范围，最上面的一行表示第二直接依赖范围，交叉单元格表示传递性依赖范围)
	
			||compile|test|provided|runtime|
			|---|---|---|---|---|
			|compile|compile|---|---|provided|
			|test|test|---|---|test|
			|provided|provided|---|provided|provided|
			|runtime|runtime|---|---|runtime|
	2. 依赖解调(详见maven实战部分)
		1. 依赖路径最近的优先
		2. 第一声明的优先
	
	3. 可选依赖（最好不要使用）
		1. 情景 A -> B，B -> X(可选)，B -> Y(可选)，项目B实现了两个特性，但是特性一依赖于X，特性二依赖与Y，但是这两个特性互斥，用户不可能同时使用两种特性。比如一个 持久层隔离工具包，他支持多种数据库，在构建工具包的时候需要多种数据库的驱动程序，但是在使用工具包的时候只会依赖一种数据库
		2. 当声明依赖为可选依赖的时候，他们只会对当前项目B产生影响，当其他项目依赖与B的时候，这些依赖不会被传递，然后在实际使用的时候，A需要显示的声明所依赖的数据库驱动
	4. 最佳实践
		1. 排除依赖
		
				<dependency>  
				    <groupId>com.gc.user</groupId>  
				    <artifactId>user-core</artifactId>  
				    <version>0.0.1-SNAPSHOT</version>  
				    <exclusions>  
				        <!-- 排除user-core模块中关于log4j的依赖 -->  
				        <exclusion>  
				            <groupId>log4j</groupId>  
				            <artifactId>log4j</artifactId>  
				        </exclusion>  
				    </exclusions>  
				</dependency>  
		2. 归类依赖(定义版本常量避免重复)

				// 设置常量
				<properties>
					<junit.version>4.12</junit.version>
				</properties>
				//使用常量
				<dependency>
					<groupId>junit</groupId>
					<artifactId>junit</artifactId>
					<version>${junit.version}</version>
					<scope>test</scope>
				</dependency>
		3. 优化依赖
			1. 查看当前项目的已解析依赖

					mvn dependency:list
			2. 查看当前项目的依赖树
			
					mvn dependency:tree
			3. 分析当期项目的依赖( 应注意 `Used undeclared dependencies` 与 `Unused undeclared dependencies` 警告 )

					mvn dependency:analyze

2. 仓库（先跳过）
	1. 依赖在仓库中的路径
		1. groupId/artifactId/version/artifactId-version.packaging
		2. 从仓库解析依赖的机制
			1. 

3. 生命周期与插件
	1. 生命周期
		1. 生命周期是为了对所有的构建过程进行抽象和统一，maven的生命周期包含了几乎所有的构建步骤(清理，初始化，编译，测试，打包，集成测试，验证，部署和站点生成)
		1. Maven拥有三套独立的生命周期
			1. clean
				* 清理项目
			2. default
				* 构建项目
			3. site
				* 建立项目站点
	2. 插件目标
		* 对于插件本身，为了能够复用代码，他往往能够完成多个任务，如果为了每个这样的功能编写一个插件显然是不可取的(如果这样会造成大量的代码重复)，于是就应该将多个功能聚集在一个插件里面，每个功能就是一个插件目标
		* 调用插件目标
		
				plugin:goal // eg dependency:list
	3. 插件绑定
		1. 具体而言是生命周期的阶段与插件的目标相互绑定，已完成某个具体的任务
		2. 内置绑定
			1. 为了让任何用户几乎不用配置就能够构建maven项目，maven在核心为一些主要的生命周期绑定了很多相应的插件，以下是内建生命周期的绑定

				![Built-in Lifecircle Bindings](http://i.imgur.com/aqUhxs1.png)
			2. 上图只列出了拥有插件绑定关系的阶段，生命周期中还有其他很多阶段默认没有绑定的任何插件，因此也没有任何实际行为
		3. 自行绑定插件

				// 将插件 maven-source-plugin 的目标 jar-no-fork 绑定到 verify 阶段
				// 当goal被绑定到了不同生命周期的阶段的时候，其执行顺序就是生命周期阶段的先后顺序
				// 当多个goal被绑定到同一个阶段的时候，插件的声明顺序决定了目标执行顺序
				<build>
				    <plugins>
				      <plugin>
				        <groupId>org.apache.maven.plugins</groupId>
				        <artifactId>maven-source-plugin</artifactId>
				        <version>2.1.1</version>
				        <executions>
				          <execution>
				            <id>attach-sources</id>
				            <phase>verify</phase>
				            <goals>
				              <goal>jar-no-fork</goal>
				            </goals>
				          </execution>
				        </executions>
				      </plugin>
				    </plugins>
			  </build>
		4. 插件配置
			1. 命令行插件配置
				1. 用户在 maven 命令中使用 -D 参数，并伴随一个 参数键=参数值 的形式配置 goal 的参数

						mvn install -Dmaven.test.skip=true
			2. POM中插件全局配置
				1. 适用于那些插件的参数值在项目创建到发布都很少改变

						// 告诉 maven-compiler-plugin 编译Java1.8的源文件，生成与 Java1.8兼容的字节码文件
						<plugin>
						  <groupId>org.apache.maven,plugins</groupId>
				          <artifactId>maven-compiler-plugin</artifactId>
				          <configuration>
				            <source>1.8</source>
				            <target>1.8</target>
				          </configuration>
				        </plugin>
			3. POM 中插件任务配置
				1. 除了为插件配置全局参数，还可以为某个插件任务配置特定的参数
				
						// 在此段配置文件中将 antrun 插件绑定到了 validate 与verify 阶段，分别在不同的阶段执行不同的任务
						// 插件全局的配置 <configuration> 元素 直接在 <build> 元素的下方，这里的 <configuration> 元素在 <execution> 下面，表示特定任务的配置
						<build>
						    <plugins>
						      <plugin>
						        <groupId>org.apache.maven.plugins</groupId>
						        <artifactId>maven-antrun-plugin</artifactId>
						        <version>1.4</version>
						        <executions>
						          <execution>
						            <id>ant-validatt</id>
						            <phase>validate</phase>
						            <goals>
						              <goal>run</goal>
						            </goals>
						            <configuration>
						              <tasks>
						                <echo> this is validate parse </echo>
						              </tasks>
						            </configuration>
						          </execution>
						          <execution>
						            <id>ant-verify</id>
						            <phase>verify</phase>
						            <goals>
						              <goal>run</goal>
						            </goals>
						            <configuration>
						              <tasks>
						                <echo> this is verify parse </echo>
						              </tasks>
						            </configuration>
						          </execution>
						        </executions>
						      </plugin>
						    </plugins>
						</build>

			4. 获取插件信息
				1. 在线插件信息
				2. 使用 `maven-help-plugin` 描述插件

						//  = 符号前后不能有空格，不然汇报错(gruopId:artifactId:version)
						mvn help:describe -Dplugin=org.apache.maven.plugins:maven-compiler-plugin:3.5.1

						// 输出
						Name: Apache Maven Compiler Plugin
						Description: The Compiler Plugin is used to compile the sources of your project.
						Group Id: org.apache.maven.plugins
						Artifact Id: maven-compiler-plugin
						Version: 3.5.1
						Goal Prefix: compiler
						
						This plugin has 3 goals:
						
						compiler:compile
						  Description: Compiles application sources
						
						compiler:help
						  Description: Display help information on maven-compiler-plugin.
						    Call mvn compiler:help -Ddetail=true -Dgoal=<goal-name> to display parameter details.
						
						compiler:testCompile
						  Description: Compiles application test sources.
			5. 从命令行调用插件
				1. maven还可以在命令行直接调用 goal，因为一些 goal 并不适合绑定到 parse，例如上面看到的 `maven-help-plugin:describe`

						mvn hepl:describe -Dplugin=compiler
						mvn dependency:tree
				2. 上面的两条命令看起来似乎有个疑问，describe 确实是goal，但是前面的 help 是什么鬼，maven是如何根据该信息找到对应版本的插件呢？
					1. maven 引入了目标前缀的概念来是命令简化，上面的 help 就是 maven-help-plugin 的目标前缀，还需要的信息（如version）将在下一节介绍

			6. 插件解析机制
				1. 插件仓库
					1. maven 会区别对待 依赖的远程仓库与 插件的远程仓库，当插件在本地仓库早不到时，maven并不会到 `<repository>`元素配置的仓库去找
					2. maven 使用 `<pluginRepositories>` 与 `<pluginRepository>` 来配置插件的远程仓库
				2. 插件的默认 groupId
					1. 在 POM 中配置的时候，如果该插件是maven官方的插件，就可以省略 groupId 的配置
				3. 解析插件版本
					1. 为了简化插件的配置与使用，在用户没有提供插件版本的时候，maven会自动解析插件版本
					2. 首先，maven在 Super POM 中为所有核心插件设置了版本
					3. 如果用户使用了某个插件是没有指定版本，但是这个插件又不是核心插件，maven会检查所有仓库中的可用版本，然后做出选择
					4. 推荐做法是 应该一直显示的设定版本,不应该依赖 maven 解析插件版本
				4. 解析插件前缀
					1. 插件前缀与 groupId:artifactId是一一对应的，这种匹配关系存储在仓库中的元数据中。这里的元数据路径是 `groupId/maven-metadata.xml`,与之前提到的元数据路径不同( `groupId/arrifactId/maven-metadata.xml` )，这里的 groupId 是啥子呢？
					2. 因为 插件主要由 Apache 与 Codehaus 的中央仓库提供，所以 maven 在解析插件仓库元数据的时候会默认使用 `org.apache.maven.plugins` 和 `org.codehaus.mojo` 两个 `groupId`
					3. 设置 setting.xml 让 maven 检查其他 groupId 上的插件

							<pluginGroups>
								<pluginGroup>com.your.plugins</pluginGroup>
							</pluginGroups>
					4. prefix 与 artifactId 的对应关系如下：

							<plugin>
						      <name>Apache Maven Compiler Plugin</name>
						      <prefix>compiler</prefix>
						      <artifactId>maven-compiler-plugin</artifactId>
						    </plugin>
							<plugin>
						      <name>Apache Maven Clean Plugin</name>
						      <prefix>clean</prefix>
						      <artifactId>maven-clean-plugin</artifactId>
						    </plugin>
					5. 总结
						* 当 `maven` 解析到如 `dependency:tree` 这样的命令之后，他首先基于默认 `groupId` 与 用户自定义的 `groupId` (请注意这里的groupId是插件的groupId)归并所有插件仓库的元数据（可能有多个插件仓库都有含有某个 groupId 的元数据），然后检查归并后的元数据，早到对应的 `artifactId`( `maven-dependency-plugins` )，然后结合当前元数据的 `groupId` (`org.apache.maven.plugins`)，最后使用上述的解析版本的方法解析到 `version` ，于是就得到了完整的插件坐标。
						* 如果在检查完所有的 `groupId` 之后都没找到插件就会报错

		1. 项目的聚合与继承
			1. 聚合与继承的关系
				1. 聚合：方便快速构建项目
				2. 继承：消除重复配置
				3. 两者的 <packaging> 都是 pom，并且两者模块中都没有实际内容
			2. 多模块项目的构建顺序
				* maven 按照 `<modules>` 中的声明顺序依次读取POM文件( 当然第一个读取的POM肯定是包含 `<modules>`的POM ) ，如果该模块没有依赖模块，那就构建该模块，否则构建起依赖的模块
				* 模块之间的依赖会形成一个有向非循环图，如果出现 A->B,B->A的情况，maven就会报错
				* 构建部分模块
						
						// 只构建 account-email 与 account-persist 模块
						mvn clean install -pl account-email,account-persist
						// 只构建 account-email 与其依赖的模块（使用 -am 选项）
						mvn clean install -pl account-email -am
						// 构建该模块与 依赖于该模块的模块列表
						mvn clean install -pl account-parent -amd
1. 使用 maven 测试
	1. maven 会使用 maven-surefire-plugin 插件来运行测试
	2. maven-surefire-plugin 的 test goal 会自动执行测试源码路径(src/test/java)下这些测试类
		1. **/Test.*.java
		2. **/*Test.java
		3. **/*TestCase.java
	3. 跳过测试

			mvn install -DskipTests
	4. 跳过测试代码的编译

			// maven.test.skip 参数同时控制 compiler 与 surefire 两个插件
			mvn install -Dmaven.test.skip=true
1. maven 灵活的构建
	1. maven 属性
		2. 内置属性
			1. `${ basedir }` 当期项目根目录
			2. `${ version }` 当前项目版本
		3. POM 属性
			1. `${ project.groupId/artifactId/version }` 项目对应名称
			2. `${ project.build.sourceDirectory }` 项目源码目录
			3. ...
		1. 自定义属性：通过 `<properties>`元素自定义属性，然后通过 **${ 属性名称 }** 来引用属性值

				<properties>
					<springfreamwork.version>4.2.6</springfreamwork.version>
				</properties>
		4. Settings 属性
			1. 引用 settings.xml 文件中 XML 元素的值
			2. `${ settings.loaclRepository }`  本地仓库的地址
		5. Java 系统属性
			1. 引用 Java系统属性
			2. `${ user.home }` 指向用户目录
			3. 可以使用 `mvn help:system` 查看全部的 Java 系统属性
		6. 环境变量属性
			1. 环境变量属性可以以 evn. 开头的 Maven 属性引用 
			2. `${ evn.JAVA_HOME }` JAVA_HOME 环境变量的值
			3. 可以使用 `mvn help:system` 查看全部的 Java 系统属性

	2. 构建环境的差异
		1. 在不同的环境中，项目源码应该使用不同的方式构建，因为在项目开发的不同阶段需要的构建配置往往是不同的，比如数据库的配置，在不同的阶段数据库名，RUL等都会变化，maven 支持针对不同的环境生成不同的构建
		2. 以数据库配置为示例
			1. 既然链接数据库的 驱动类，URL等属性都会变化，那么我们先定义一些 Maven 属性
				1. database.jdbc.driverClass = ${ db.driver }
				2. database.jdbc.connectionURL = ${ db.url }
				3. database.jdbc.username = ${ db.username }
				4. database.jdbc.password = ${ db.password }
			2. 然后在一个 `profile` 里面定义这些变量

					<profiles>
						<profile>
							<id>dev</id>
							<properties>
								<db.driver>com.jdbc.Driver</db.driver>
								<db.url>jdbc:mysql://127.0.0.1:3306/test</db.url>
								<db.username>dev</db.username>
								<db.password>dev_pass</db.username>
							</properties>
						</profile>
					</profiles>
			3. 但是还是会存在一个问题就是,maven 属性（如 ${db.username}）只会在 POM 文件中才会被解析成对应的值（dev），当被放到 src/main/resources/ 目录下的文件时，便还会是 ${db.username}，需要解决这个问题就要让插件能够解析资源文件中的 maven 属性，即开启资源过滤

					<filtering>true</filtering> // 开启资源过滤
		2. 激活 profile
			1. 命令行激活

					mvn install -Pdev // -P 跟需要激活的profile名称，多个profile以逗号隔开
			2. settins 文件显示激活
				1. 应用场景：希望某个 profile 默认一直处于激活状态
				2. 配置 `settings.xml` 文件中的 `activeProfile` 元素

						<activeProfiles>
							<activeProfile> profile 名 </activeProfile>
						</activeProfiles>
		3. 系统属性激活
			1. 用户可以配置当某个系统属性存在时激活某个 profile

					// 当存在系统属性 test 时，激活此 profile
					<profiles>
						<profile>
						    ...
							profile 其他属性
							...
							<activation>
								<property>
									<name>test</name>
									// 这句表示当系统属性 test=x 时激活此 profile
									// <value>x</value>
								</property>
							</activation>
						</profile>
					</profiles>

					// 当需要指定系统属性的值得时候可以这样做
					mvn install -Dtest=x
			4. 根据操作系统环境激活
				1. 通过配置 <activation> 元素下 <os> 元素

			5. 文件存在与否激活

					<activation>
						<file>
							<missing>x.properties</missing>
							<exists>y.properties</exists>
						<file>
					</activation>
			6. 默认激活
				1. 通过定义 profile 的时候添加元素 <activeBydefault> 为 true，来指定 profile，自动激活
				2. 但是如果 POM 中有任何一个 profile 通过以上方式被激活，默认激活配置都会失效

			7. 了解当前激活的 profile

					mvn help:active-profiles
			6. 列出当前所以的 profile

					mvn help:all-profiles
	3. profile 的种类
		1. pom.xml
			* pom.xml 中声明的 profile 仅仅对当期项目有效
		2. 用户 settings.xml
			* ${user}/.m2/settings.xml 中的 profile 对该用户的所有 maven 项目有效
		3. 全局 settings.xml
			* MAVEN_HOME/conf/settings.xml 对本机所有 maven 项目有效
		4. profiles.xml
			* 特性已经移除，不再使用
		5. 各种 profile 可以声明的元素，因为只有 pom 中的 profile 能够随项目一起发布
			1. 在 pom.xml 中

					<project>  
					    <repositories></repositories>  
					    <pluginRepositories></pluginRepositories>  
					    <distributionManagement></distributionManagement>  
					    <dependencies></dependencies>  
					    <dependencyManagement></dependencyManagement>  
					    <modules></modules>  
					    <properties></properties>  
					    <reporting></reporting>  
					    <build>  
					        <plugins></plugins>  
					        <defaultGoal></defaultGoal>  
					        <resources></resources>  
					        <testResources></testResources>  
					        <finalName></finalName>  
					    </build>  
					</project>
			2. 在其他 profile 中
			
					<project>  
					    <repositories></repositories>  
					    <pluginRepositories></pluginRepositories>  
					    <properties></properties>  
					</project>
	4. web 资源过滤
	5. 在 profile 里面激活集成测试

1. 编写 maven 插件
2. 编写 Archetype
	1. Archetype 项目内容
		1. pom.xml  // Archetype 自生的 POM
			1. 需要指定 packaging 类型为 POM，不然编译会出错
		2. src/main/resources/archetype-resources/pom.xml  // 基于该 Archetype 生成的项目的POM原型
		3. src/main/resources/META-INF/maven/archetype-metadata.xml // Archetype 的描述文件
			1. 声明那些目录及文件应该包含在 Archetype 中
			2. 这个 Archetype 包含的属性参数
		4. src/main/resources/archetype-resources/ ** 
	2. 实例

			<?xml version="1.0" encoding="UTF-8"?>
			<archetype-descriptor name="sample">
				<fileSets>
					<fileSet filtered="true" packaged="true">
						<directory>src/main/java</directory>
						<includes>
							<include>**/*.java</include>
						</includes>
					</fileSet>
					<fileSet filtered="true" packaged="true">
						<directory>src/test/java</directory>
						<includes>
							<include>**/*.java</include>
						</includes>
					</fileSet>
					<fileSet filtered="true" packaged="false">
						<directory>src/main/resources</directory>
						<includes>
							<include>**/*.properties</include>
						</includes>
					</fileSet>
				</fileSets>
				<requiredProperties>
					<requiredProperty key="port" />
					<requiredProperty key="groupId">
						<defaultValue>com.juvenxu.mvnbook</defaultValue>
					</requiredProperty>
				</requiredProperties>
			</archetype-descriptor>
		1. fileSet 表示一个目录
			1. filterd 属性表示是否对该文件集合 应用属性替换
			2. packaged 属性表示是否将给路径下的内容放到生成项目的包路径下
				1. 就是如果 package=true，directory=x，输入的 package=y，那么 x 目录下的全部文件会被复制到 y 目录下
		2. 注意 在需要使用我们定义的 archetype 创建工程的时候必须先 install 当前工程，不然执行命令会出错
	2. Archetype Catalog
		1. 当用户不指定 `Archetype` 坐标的方式使用 Archetype 插件的时候，会得到一个 `Archetype` 列表供选择，这个列表信息来自于 `archetype-catalog.xml` 的文件，现在我们来把我们自定义的 `archetype` 加入到此列表中
		2. Archetype Catalog 的来源
			1. internal : archetype 插件内置的 Archetype Catalog
			2. local : 用户本地的 Archetype Catalog，其路径为 ~/.m2/archetype-catalog.xml  //该文件默认不存在
			3. remote : maven 中央仓库的 Archetype Catalog
			4. file : //... 用户可以指定本地任何地方的 archetype-catalog.xml 文件
			5. http : //... 使用http协议指定远程的 archetype-catalog.xml 文件

					// 使用 archetypeCatalog 选项指定 archetype-catalog 文件路径
					// 当需要指定多个 archetype-catalog 文件来源的时候用 , 分隔
					mvn archetype:generate -DarchetypeCatalog=file:// /tem/archetype-catalog.xml,local,remote

1. POM 元素参考
	1. `<parent>`
	2. `<modules>`
	3. `<packaging>`
	4. `<description>`
	5. `<scm>`
	6. `<prerequisites> < maven>`   // maven 版本最低要求
	7. `<build> <filters> <filter>` //通过 properties 文件定义资源过滤属性
	8. `<build> <extensions> <extension>` // 拓展 maven 的核心
	9. `<build> <pluginManagement>` // 插件管理
	10. `<dependencyManagement>` // 依赖管理
	10. `<build> <plugins> <plugin>`  // 插件
	11. `<profiles> <profile>` // POM profile
	12. `<distributionManagement> <repository>`
	13. `<repositorys> <repository>` // 仓库
	14. `<pluginRepositories> <pluginRepository>` // 插件仓库
	15. `<properties>` // maven 属性
