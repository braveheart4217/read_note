## 解决调试JDK源码无法查看局部变量的方法

###一.问题阐述
首先我们要明白JDK source为什么在debug的时候无法观察局部变量，因为在jdk中,sun对rt.jar中的类编译时,去除了调试信息,这样在eclipse中就不能看到局部变量的值。这样的话，如果在debug的时候查看局部变量，就必须自己编译相应的源码使之拥有调试信息。

###二.解决方案
选择或创建你的工作目录，比如我选择：F:\

1. 在你的工作目录中，创建文件夹jdk_src，用于存放源码,创建文件夹jdk_debug，用于输出编译结果。
2. 从你的机器装jdk的地方找到src.zip，在你得JDK_HOME可以找到，比如我的在`C:\Program Files\Java\jdk1.8.0_65`目录下。然后把src解压到jdk_src目录下。
3. 解压完后，选择你需要编译的内容，删除剩下的。一般选择如下的几个文件夹就可以了：`java,javax,org` 这三个目录就可以了
4. 从你得 `JDK_HOME\jre\lib` 中将 `rt.jar` 复制到你的工作目录中，这样做的目的可以让你减少在命令行中输入的文件名。
5. 执行如下这条命令：
	
		dir /B /S /X jdk_src\*.java > filelist.txt
	会创建创建一个叫做filelist.txt的文件，这个文件存放了所有你将要编译的类的名称。
6. 然后执行如下的命令：

		javac -J-Xms16m -J-Xmx1024m -sourcepath F:\jdk_src -cp F:\rt.jar -d F:\jdk_debug -g @filelist.txt >> log.txt 2>&1  

	这条命令将要编译所有你指定的文件，并把编译结果输出到`jdk_debug`目录中，同时产生`log.txt`日记文件。这个日记文件记录着编译警告，但是没有错误。
7. 进入到jdk_debug目录中，输入如下命令：
		
		jar cf0 rt_debug.jar *  

	这个命令可以生成我们需要的`rt.jar`带编译信息的jar包。

8. 把这个生成的`rt_debug.jar`包复制到`JDK_HOME\jre\lib\endorsed`。如果没有`endorsed`目录，自己创建一下。
9. 如果你是在`eclipse`中`debug`,点击`Window->Prefences->Java->Installed JRES`，选择相应的JDK,点击Edit,然后选择点击`Add External jars`，选择我们上面生成的`rt_debug.jar`，就可以了。
10. 现在完成了所有的步骤了，赶快尝试debug一下，如果可以查看局部变量了，那么恭喜你成功了。


注：文章并不是原创，参考如下链接，并修改了一点原文作者留下的一点笔误

参考链接：

[解决调试JDK源码无法查看局部变量的方法](http://blog.csdn.net/appleprince88/article/details/21873807)

[debug jdk source can't watch variable what it is](http://stackoverflow.com/questions/18255474/debug-jdk-source-cant-watch-variable-what-it-is)
