###Linux开机进入命令行界面
每个版本的Linux开机进入命令行的方式都不同，现在介绍一下各个版本的方法：

**Ubuntu**

在**Ubuntu 15**之前可以使用如下方式：

		sudo gedit /etc/default/grub
找到这一行

		GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
修改为

		GRUB_CMDLINE_LINUX_DEFAULT="quiet splash text"
然后更新grub

		sudo update-grub

然后在**Ubuntu 15**之后如果还是使用上述命令会看到提示说，此种方式不再支持，所以新的方式是：

		sudo systemctl set-default multi-user.target
然后如果想进入图形界面客户使用
		Sudo service lightdm start
或者

		startx


**Centos**

在**Centos7**之前可以修改**/etc/inittab**文件，将文件中的

		:id:5:initdefault:
改为

		:id:3:initdefault:
即将默认的runlevel由5改为3，但在**CentOS7**下打开**/etc/inittab**页面显示如下：

		# inittab is no longer used when using systemd.
		#
		# ADDING CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
		#
		# Ctrl-Alt-Delete is handled by /etc/systemd/system/ctrl-alt-del.target
		#
		# systemd uses 'targets' instead of runlevels. By default, there are two main targets:
		#
		# multi-user.target: analogous to runlevel 3
		# graphical.target: analogous to runlevel 5
		#
		# To set a default target, run:
		#
		# ln -sf /lib/systemd/system/<target name>.target /etc/systemd/system/default.target
		#

提示说inittab文件已经不被使用，现在CentOS7使用systemd作为新的init系统，而systemd系统使用“target”来代替“runlevel”，默认有两个主要的target：

    	multi-user.target：相当于runlevel 3[命令行界面]
		graphical.target ：相当于runlevel 5[图形界面]

设置默认的target则使用命令：

     	ln -sf /lib/systemd/system/<target name>.target /etc/systemd/system/default.target
 
我这里要将CentOS7开机默认进入命令行界面，则运行命令：
     
		ln -sf /lib/systemd/system/multi-user.target /etc/systemd/system/default.target
或者

		systemctl set-default multi-user.target(命令模式)
		systemctl set-default graphical.target(图形模式)


**Fodora**最近没有怎么使用了，然后如果有需要的可以自行Google

参考链接：

[Centos7开机启动命令行](https://segmentfault.com/q/1010000000705452)

[Ubuntu15开机启动命令行](http://www.linuxdiyf.com/linux/16802.html)

[Ubuntu12启动到命令行](http://my.oschina.net/leopardsaga/blog/109984?fromerr=9cXhzUkp)