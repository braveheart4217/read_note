##maven实战
出现的问题：

1. 需要了解的节点的含义：
	1. packaging
	2. dependencies
	3. dependencymanagement
	4. prooerties
	5. profile
	6. modules
	7. module
	8. build
	9. plugin

1. Maven坐标包括 groupId,artifactId,version,packing,class-sifier
	1. 其中 classifier 用来帮助定义构建输出的一些附属构建

1. 引入了依赖的Jar包，没被管理起来
	1.  理解 `<dependencies>` 节点与 `<dependencymanagement>` 节点
		1. `<dependencies>` 节点项目依赖，项目所依赖的Jar包
		2. `<dependencymanagement>`里面的 `<dependencies>` 集中管理依赖版本,但是不添加依赖关系
		3. 就是说 `<dependencymanagement>` 主要是为了**统一多模块项目之间各个项目依赖的版本**，但是并没有为你的哪个模块添加依赖，依赖还是需要你自己在模块 `POM` 的 `<dependencies>` 节点手动添加，只不过可以不用填写版本值，因为版本值已经在 父 `POM` 指定
		4. 只有当模块的 `<dependency>` 元素中没有指明版本信息时， `<dependencyManagement>` 中的 `<dependency>` 元素才起作用
		5. 而父 `POM` 声明的 `dependencies`元素即使在子项目中不写该依赖项，那么子项目仍然会从父项目中继承该依赖项
		6. `import` 范围依赖
			1. 仅仅在 `<dependencyManagement>` 下才有效
			2. 使用该范围依赖通常指向一个POM，作用是将目标POM中的 `<dependencyManagement>` 元素 导入并合并到当前POM
			3. **应用情景**：如果10个、20个甚至更多模块继承自同一个模块，那么按照我们之前的做法，这个父模块的 `dependencyManagement` 会包含大量的依赖。如果你想把这些依赖分类以更清晰的管理，那就不可能了，`import scope` 依赖能解决这个问题。你可以把 `dependencyManagement` 放到单独的专门用来管理依赖的POM中，然后在需要使用依赖的模块中通过 `import scope` 依赖，就可以引入 `dependencyManagement`
			4. 可以为每一个模块都创建一个管理该模块的 `<dependencyManagement>` POM，然后在父模块中使用 import scope 导入该模块的 `dependencyManagement` POM，这样父 POM 中放入 `dependencyManagement` 就会干净很多

					// 将 account-parent 中 dependencyManagement 导入合并到当前POM
					<dependency>
				        <groupId>com.junx.account</groupId>
				        <artifactId>account-parent</artifactId>
				        <version>1.0.SNAPSHOT</version>
						<type>pom</type>
				    	<scope>import</scope>
				    </dependency>

		7. `<pluginManagement>` 管理插件，其作用与 `<dependencyManagement>` 类似
			1. 只有那些普适的插件配置才应该使用 `pluginManagement` 提取到父POM中，因为不同模块中的插件的配置往往不一样
			2. 在该元素中的配置往往不会造成实际的插件调用，只有当 POM 中配置了真正的 plugin 元素，并且 groupId 与 artifactId 与 `pluginManagement` 中相同的时候，`pluginManagement` 中的配置才会生效
			3. 同样子模块也可以覆盖父模块中对于 plugin 的配置

2. 两个 WAR 包的调用
	1. `<dependency>` 节点下面的<type> 属性指明为 `war`

			<dependency>  
            	<groupId> </groupId>  
            	<artifactId> </artifactId>  
        		<version> </version>      
            	<type>war</type>  
   			</dependency>
3. 依赖传递
	1. 在工程的依赖树上，深度越浅，越被优先选择
    2. 若两个依赖包处于依赖树上的同一层，则谁在前选择谁
    3. 解释：
	    1. `User-core` 依赖的 `Commons-logging` 的版本号为1.0.4
	    2. `User-log` 依赖的 `Commons-logging` 的版本号为1.1.1
	    3. `User-service` 依赖与 `User-core` 与 `User-log`
	    4. 如果 `User-core` 与 `User-log` 都是直接依赖于 `Commons-logging` (也就是说其在 **依赖树上的深度** 是一样的)，那么 `User-service` 里面 `Commons-logging`版本取决于 `User-service` 对 `User-core` 与 `User-log` 的依赖声明的次序(就是谁声明在前面就使用谁的依赖)
	    5. 但是如果 `User-core` 对于 `Commons-logging` 的依赖并不是直接依赖，就是说他可能依赖于 `dbunit`，然后`dbunit` 对 `Commons-logging` 有依赖，那么 `User-core` 与 `User-log` 对于 `Commons-logging` 的依赖深度不同，并且是 `Use-core` > `User-log`，这种情况下会选择依赖深度浅的依赖

2. `<dependency>` 全貌

		<dependency>
			<groupId></groupId>
			<artifactId></artifactId>
			<version></version>
			<type></type>
			<scope></scope>       // 依赖的范围
			<optional></optional> // 标记是否可选
			<exclusions>          // 用来排除传递性依赖
				<exclusion>
				...
				</exclusion>
			</exclusions>
		</dependency>