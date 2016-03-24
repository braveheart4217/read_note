##Log4j
1. 基础
	
	* 核心对象
		* `Logger`
			* 最上层的对象，负责获取日志信息，并存储在一个命名空间中
		* `Layout`
			* 格式化日志信息的对象，在发布信息之前，为appender对象提供支持
			* 其使日志变得可读，可复用
		* `Appender`
			* `APpender` 负责将日志信息发布到不同的地方，如数据库，console，文件
		* `Level`
			* `Level` 对象定义了日志信息的粒度与优先级
			* API定义了七种级别：OFF，DEBUG，INFO，ERROR，WARN，FATAL，ALL
		* `Filter`
			* `Filter`用来对日志进行分析，进而决定该日志是否被记录
			* `一个Appender` 对象可以对应多个Filter对象，当日志信息传给 Appender 对象时，与其关联的所以、filter需要判断是否需要将该日志发布到目的地
		* `ObjectRender`
			* 负责为传入日志框架的不同对象提供字符串的表示，Layout对象使用该对象准备最终的日志信息
		* `LogManager`
			* 负责从系统级的配置文件或者类中读取初始配置信息
	
		![整体框架](http://i.imgur.com/xxso77o.jpg)

2. Log4j的配置
	* `Log4j` 的配置包括在配置文件中指定 `Level`、定义 `Appender` 和指明 `Layout`
	* `Log4j.properties` 文件是 `Log4j` 的配置文件，属性以键值对的形式定义
	* `Log4j.properties` 示例
			
			# Define the root logger with appender X
			Log4j.rootLogger = DEBUG, X

			# Set the appender named X to be a File appender
			Log4j.appender.X=org.apache.Log4j.FileAppender

			# Define the layout for X appender
			Log4j.appender.X.layout=org.apache.Log4j.PatternLayout
			Log4j.appender.X.layout.conversionPattern=%m%n

		* 日志级别定义为 DEBUG，并将名为 FILE 的 apppender 添加上
		* `appender` FILE 定义为 `org.apache.Log4j.FileAppender`，它将日志写入 log 目录下一个名为 log.out 的文件中
		* `layout` 被定义为 `%m%n`，即打印出来的日志信息末尾加入换行上

3. Appender 主要属性
	
	|属性|描述|
	|--|--|
	|layout| Appender使用Layout对象和与之关联的模式来格式化日志信息|
	| target | 目的地可以是console，文件，或者依赖于 appender 的对象|
	| Level| 级别用来控制过滤日志信息|
	| threshold |Appender可脱离日志级别定义的一个阈值级别，Appender 对象会忽略所有级别低于阀值级别的日志
	|Filter| Filter对象可在级别基础之上分析日志信息，来决定 Appender 对象是否 处理或者忽略一条日志记录
	* 将 Appender 对象添加到 Logger 对象

			Log4j.logger.[logger-name]=level, appender1,appender..n
	* 或者在XML中配置
		
			<logger name="com.apress.logging.Log4j" additivity="false">
   				<appender-ref ref="appender1"/>
   				<appender-ref ref="appender2"/>
			</logger>






参考链接

* [Log4j教程](http://wiki.jikexueyuan.com/project/log4j/architecture.html)