#安装Jetty服务器在Contos7

Jetty is a pure Java-based HTTP (Web) server and Java Servlet container. Jetty is now often used for machine to machine communications, usually within larger software frameworks. But the other Web Servers are usually associated with serving documents to humans. Jetty is developed as a free and open source project as part of the Eclipse Foundation. The web server is used in products such as Apache ActiveMQ, Alfresco, Apache Geronimo, Apache Maven, Apache Spark, Google App Engine, Eclipse, FUSE, Twitter’s Streaming API and Zimbra.

This article explains ‘How to install jetty web server in your CentOS server’.



1. First of all we have to install java JDK, By the following command:

		yum -y install java-1.7.0-openjdk wget


2. After the JDK installation, We will download the latest version of Jetty:

		wget http://download.eclipse.org/jetty/stable-9/dist/jetty-distribution-9.2.7.v20150116.tar.gz
3. Extract and move the the downloaded package to /opt:

		tar zxvf jetty-distribution-9.2.5.v20141112.tar.gz -C /opt/
4. Rename the file name to jetty:

		mv /opt/jetty-distribution-9.2.5.v20141112.tar.gz/ /opt/jetty
5. Create a user called jetty:

		useradd -m jetty
6. Change the ownership of jetty:

		chown -R jetty:jetty /opt/jetty/
7. Make a Symlink jetty.sh to /etc/init.d directory to create a start up script file:

		ln -s /opt/jetty/bin/jetty.sh /etc/init.d/jetty
8. Add script:

		chkconfig --add jetty
9. Make the jetty web server auto starts on system boot:

		chkconfig --level 345 jetty on
10. Open /etc/default/jetty in your favorite editor and replace port and listening address desired value:（这个文件本来就不存在）

		vi /etc/default/jetty
		JETTY_HOME=/opt/jetty
		JETTY_USER=jetty
		JETTY_PORT=8080
		JETTY_HOST=50.116.24.78
		JETTY_LOGS=/opt/jetty/logs/
We finished the installation, Now you have to start the jetty service.

		service jetty start

Now you can access jetty web sever in  http://<youripaddress>:8080

参考链接：

[安装Jetty服务器在Contos7](http://www.unixmen.com/install-jetty-web-server-centos-7/)