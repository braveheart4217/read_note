#Linux 为用户设定、修改密码 passwd

今天来介绍一下如何使用 passwd 命令来为用户设定密码，及修改密码。

passwd 语法结构：


	NAME
	passwd – update a user’s authentication tokens(s)

	SYNOPSIS
	passwd [-k] [-l] [-u [-f]] [-d] [-n mindays] [-x maxdays] [-w warndays] [-i inactivedays] [-S]
	[–stdin] [username]

###passwd 参数浅谈：

	参数	描述
	-l	锁定已经命名的账户名称，只有具备超级用户权限的使用者方可使用。
	-u	解开账户锁定状态，只有具备超级用户权限的使用者方可使用。
	-x, –maximum=DAYS	多少天之后必须更换密码，也就是说这个密码最大使用多少天，只有具备超级用户权限的使用者方可使用。
		对应 /etc/shadow 的第五个字段。
	-n, –minimum=DAYS	多少天内不能更换密码，也就是说这个密码最少得用多少天，只有具备超级用户权限的使用者方可使用。
		对应 /etc/shadow 的第四个字段。
	-d	删除使用者的密码, 只有具备超级用户权限的使用者方可使用。
	-S	检查指定使用者的密码认证种类, 只有具备超级用户权限的使用者方可使用。
###passwd 使用规则：

我们都知道，在 Linux 里，“root”账号绝对是一个大BOSS，其有着至高无上的权力。呵呵。另外昨天咱们也看到了，新账号都是由　root 权限建立的，那么自不必说，如果用户的密码不见了，就可以让 root 去帮助修改密码，而 root 是不用知道你原来密码的。另外，也只有 root 可以随便设置密码，即使该密码并不符合系统的密码要求。

另外，用户是可以更改自己的密码的，这时 passwd 后面是不用跟用户名的。不过普通用户就必须得遵守一定的规则了，创建密码的规则文件是： /etc/pam.d/passwd ，其内设定了一些设置 Linux 用户密码的规则。

一般来说，输入的密码尽量要符合如下要求：

密码不能与账号名称相同
密码尽量不要选用英文单词这种字典里面会出现的字符串
密码需要超过8个字符
passwd 使用示例：

口说无凭，咱也为我昨天创建的几个垃圾账号来设置密码。呵呵。

### root 创建密码，不用遵守啥规则，直接改即可
	[simaopig@xiaoxiaozi pam.d]$ su
	密码：
	[root@xiaoxiaozi pam.d]# passwd chongpig
	更改用户 chongpig 的口令 。
	新的 密码：
	重新输入新的 密码：
	无效的密码： 它基于字典单词
	无效的密码： 过于简单
	passwd： 所有的身份验证令牌已经成功更新。
	[root@xiaoxiaozi pam.d]# passwd chongpig
	更改用户 chongpig 的口令 。
	新的 密码：
	重新输入新的 密码：
	无效的密码： 它基于字典单词
	无效的密码： 过于简单
	passwd： 所有的身份验证令牌已经成功更新。
	[root@xiaoxiaozi pam.d]# grep chongpig /etc/shadow
	chongpig:$6$mVu5nBAo$4HzNf92n3RYAiDfVk2Q5YtIxfjrVxvYHAusGeUAWfabLr5WIuQdn/2HCcsHwKRoRpxGLCJ.wA.1QLgG.rXuhw/:14447:0:99999:7:::

虽然告诉我密码太简单，虽然告诉我说密码基于单词，不过咱怕啥？咱是 root 啊，改之，列为可以看一下，现在 chongpig 账号是已经可以用密码登录了的，其在 /etc/shadow 内的密码栏也有有值的了，而不再是 !!


	# 更改当前用户密码，非roto账号
	[simaopig@xiaoxiaozi ~]$ passwd 
	更改用户 simaopig 的口令 。
	为 simaopig 更改 STRESS 密码。
	（当前）UNIX 密码：
	新的 密码：
	重新输入新的 密码：
	密码未更改

	[simaopig@xiaoxiaozi ~]$ passwd
	更改用户 simaopig 的口令 。
	为 simaopig 更改 STRESS 密码。
	(当前)UNIX 密码：
	新的 密码：
	重新输入新的 密码：
	passwd： 所有的身份验证令牌已经成功更新。

唠叨、总结：

如何创建新用户，如何为用户设定、修改密码都介绍过了。明天应该会为大家介绍一下更改用户信息的方法，希望大家喜欢。

**PS**：今天的日全食一点感觉都没有，因为北京是阴天并且晚上还下起了雨，当时我和丫头决定去吃烤鱼，但是下雨打电话一问，人家已经弄上了。无奈啊。不过幸好在下班时雨停了，开心。呵呵。

参考链接：

[passwd的使用](http://www.xiaoxiaozi.com/2009/07/22/1219/)