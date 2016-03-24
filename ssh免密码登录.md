###SSH免密码登录
在每次使用ssh登录的时候要输入密码是件很麻烦的事情，那么今天我们就来解决一下这个问题。
解决这个问题的原理就是用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录shell，不再要求密码。

这种方法要求用户必须提供自己的公钥。如果没有现成的，可以直接用ssh-keygen生成一个：

	$ ssh-keygen
生成之后就要将本机的公钥添加到目标机的 **~/.ssh/authorized_keys** 文件中，这时候有两个方法，一种是用过**scp**，此文件传输到目标机，然后在目标机将文件内容重定向到上述文件中

    cat id_rsa.pub >> ~/.ssh/authorized_keys
但是此种方式比较麻烦，简单一点的这时再输入下面的命令，将公钥传送到远程主机host上面：

	$ ssh-copy-id user@host
如果有问题可以从以下几个方面着手解决
>1、Linux防火墙是否关闭 
>
>2、编辑目标机的 **/etc/ssh/sshd_config** 这个文件，去掉下面几行前面"#"注释，然后重启ssh

		RSAAuthentication yes
		PubkeyAuthentication yes
		AuthorizedKeysFile .ssh/authorized_keys

		service sshd restart
然后应该就没有什么问题了。

还有实际过程中可能会遇到需要多台机器都要不输入密码来登录目标服务器，这时候可以自己写个脚本来解决

		#/bin/bash
		ssh-keygen -t rsa #生成RSA密钥
		ssh-copy-id happyelement1@10.130.140.239 #将密钥添加到目标主机

然后可能还需要通过 **scp** 传输多个文件或者文件夹到目标机，然后还有一段shell代码，
		<#/bin/bash

		cmd="scp"
		option="-r"
		file=0
		dir=0
		host="username@host:path"
		path=$1;

		if [ -z $path ]
		then 
			echo "the path cannot be empty!"
			exit
		fi	 

		if [ -f "$path" ]
		then
			option=""
			file=1;
		fi

		if [ -d "$path" ]
		then
			dir=1
		fi

		if [ $dir == 0 -a $file == 0 ]
		then
			echo "the dir or file does not exist!"
			exit
		fi
		cmd $option $path $host>



参考链接：

[SSH原理与应用](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)

[SSH自动登录的几种方法](http://blueicer.blog.51cto.com/395686/88175)