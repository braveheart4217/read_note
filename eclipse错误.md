### Eclipse错误：Problems opening an editor Reason: [Project Name] does not exist解决办法
起因：最近直接从github 上下载一个demo到电脑，用eclipse打开之后，发现按F3时候出现如上所述的错误提示，于是百度之后发现解决方法如下（仅针对maven项目）

现在电脑上安装maven程序，也可以不安装，在eclipse下也可直接执行，具体方法自行百度

进入项目所在的目录，执行如下命令

	mvn eclipse:eclipse
然后在eclipse下刷新即可