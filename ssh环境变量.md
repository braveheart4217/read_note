##SSH连接远程主机执行脚本的环境变量问题
近日在使用ssh命令ssh user@remote ~/myscript.sh登陆到远程机器remote上执行脚本时，遇到一个奇怪的问题：

	~/myscript.sh: line n: Java: command not found

`Java` 明明已通过 `/etc/profile` 配置文件加到环境变量中，但这里为何会找不到？如果直接登陆机器 `remote` 并执行 `~/myscript.sh` 时，脚本可以找到并顺利执行。但为什么使用了 `ssh` 远程执行同样的脚本就出错了呢？两种方式执行脚本到底有何不同？如果你也心存疑问，请跟随我一起来展开分析。

##Bash的四种模式
在 man page 的 `INVOCATION` 一节中讲述了 bash 的四种模式，bash 会根据这四种模式而选择加载不同的配置文件，而且的加载的顺序也不同。本文ssh问题的答案就存在于这几种模式当中，所以在我们揭开谜底之前先来分析这些模式

*  `interactive + login shell`
	*  交互式的登录shell，需要解释两个名词，`interactive` 和 `login`
	*  `login` ，即登录，`login shell` 是指用户以非图形化界面或者以ssh登陆到机器上时获得的第一个 shell，简单些说就是需要输入用户名和密码的 shell。因此通常不管以何种方式登陆机器后用户获得的第一个 shell 就是 login shell，**强调有登陆这个过程**
	*  `interactive` 意为交互式，`interactive shell` 会有一个输入提示符，并且它的标准输入、输出和错误输出都会显示在控制台上。所以一般来说只要是需要用户交互的，即一个命令一个命令的输入的 shell 都是 `interactive shell`。而如果无需用户交互，它便是 `non-interactive shell`。通常来说如 `bash script.sh`  此类执行脚本的命令就会启动一个 `non-interactive shell`，它不需要与用户进行交互，执行完后它便会退出创建的 shell
	*  那么此模式最简单的两个例子为：
		* 用户直接登陆到机器获得的第一个 shell
		* 用户使用 `ssh user@remote` 获得的 shell
	* 加载配置文件
		* 这种模式下，shell首先加载/etc/profile，然后再尝试依次去加载下列三个配置文件之一，一旦找到其中一个便不再接着寻找：
			* ~/.bash_profile
			* ~/.bash_login
			* ~/.profile
	* 为了验证这个过程，我们来做一些测试
		* 首先设计每个配置文件的内容如下：

				user@remote > cat /etc/profile
				echo @ /etc/profile
				user@remote > cat ~/.bash_profile
				echo @ ~/.bash_profile
				user@remote > cat ~/.bash_login
				echo @ ~/.bash_login
				user@remote > cat ~/.profile
				echo @ ~/.profile
		*  打开一个 `login shell`
			
				bash -l
		* 进入这个新的login shell，便会得到以下输出：

				@ /etc/profile
				@ /home/user/.bash_profile
		* 如果依次移除 `.bash_profile , .bash_login`，可以验证此三个文件的加载次序为:
			* `~/.bash_profile > ~/.bash_login > ~/.profile`

*  `non-interactive login shell`
	*  非交互式的登陆shell，这种是不太常见的情况。一种创建此shell的方法为：
	
			bash -l script.sh

	* 前面提到过-l参数是将shell作为一个 `login shell` 启动，而执行脚本又使它为 `non-interactive shell`
	* 对于这种类型的shell，配置文件的加载与第一种完全一样

* `interactive non-login shell`
	* 交互式的非登陆shell，这种模式最常见的情况为在一个已有 `shell` 中运行 `bash`，此时会打开一个交互式的 `shell`，而因为不再需要登陆，因此不是`login shell`
	* 加载配置文件
		* 对于此种情况，启动 `shell` 时会去查找并加载 `/etc/bash.bashrc` 和 `~/.bashrc` 文件
	* 文件加载测试
		* 设计各配置文件内容如下

				user@remote > cat /etc/bash.bashrc
				echo @ /etc/bash.bashrc
				user@remote > cat ~/.bashrc
				echo @ ~/.bashrc
		* 启动一个交互式的非登录 shell，输出为
			
				@ /etc/bash.bashrc
				@ /home/user/.bashrc
		* 加载顺序为 `/etc/bash.bashrc`  >  `~/.bashrc`
	* `bashrc VS profile`
		* 从刚引入的两个配置文件的路径可以很容易的判断，第一个文件是全局性的，第二个文件属于当前用户
		* 此处的文件和前面模式中的文件的区别
			* 第一种模式中的 `profile` 类型文件，它是某个用户唯一的用来设置全局环境变量的地方, 因为用户可以有多个 `shell` 比如 `bash, sh, zsh` 等, 但像环境变量这种其实只需要在统一的一个地方初始化就可以, 而这个地方就是 `profile`，所以启动一个 `login shell` 会加载此文件，后面由此 `shell` 中启动的新 `shell` 进程如 `bash，sh，zsh` 等都可以由 `login shell` 中继承环境变量等配置
			* `bashrc` 的其后缀 `rc` 的意思为 `Run Commands`，由名字可以推断出，此处存放 bash 需要运行的命令，但注意，这些命令一般**只用于**交互式的shell，通常在这里会设置交互所需要的所有信息，比如 bash 的补全、alias、颜色、提示符等等
			* 故引入多种配置文件完全是为了更好的管理配置，每个文件各司其职，只做好自己的事情

* `non-interactive non-login shell`
	* 非交互非登陆的shell，创建这种shell典型有两种方式，其都是创建一个shell，执行完脚本之后便退出，不再需要与用户交互
		* `bash script.sh`
		* `ssh user@remote command`
	* 加载配置文件
		* 其会去寻找环境变量 `BASH_ENV`，将变量的值作为文件名进行查找，如果找到便加载它

* 直观的视图
	* bash的每种模式会读取其所在列的内容，首先执行A，然后是B，C。而B1，B2和B3表示只会执行第一个存在的文件


		|                | `login`<th width= 10>  |`interactive` <br> `non-login` <th width= 10>|`non-interactive` <br>`non-login`|
		|----------------|:--------|:-----------|:---------------|
		|`/etc/profile`    |   A    |           |               |
		|`/etc/bash.bashrc`|        |    A      |               |
		|`~/.bashrc `      |        |    B      |               |
		|`~/.bash_profile` |   B1   |           |               |
		|`~/.bash_login`   |   B2   |           |               |
		|`~/.profile`      |   B3   |           |               |
		|`BASH_ENV`        |   <th width= 10>     |    <th width= 10>       |   A       |

	* 更加直观的图示

		![bash 文件记载流程](http://www.solipsys.co.uk/images/BashStartupFiles1.png)
	* 上图的情况稍稍复杂一些，因为它使用了几个关于配置文件的参数：`--login，--rcfile，--noprofile，--norc`，这些参数的引入会使配置文件的加载稍稍发生改变，不过总体来说，不影响我们前面的讨论，相信这张图不会给你带来更多的疑惑

* 典型模式总结
	* 下面我们对一些典型的启动方式各属于什么模式进行一个总结：
		* 登陆机器后的第一个 `shell：login + interactive`
		* 新启动一个 shell 进程，如运行 `bash：non-login + interactive`
		* 执行脚本，如 `bash script.sh：non-login + non-interactive`
		* 运行头部有如 `#!/usr/bin/env bash` 的可执行文件，如 `./executable：non-login + non-interactive`
		* 通过 ssh 登陆到远程主机：`login + interactive`
		* 远程执行脚本，如 `ssh user@remote script.sh：non-login + non-interactive`
		* 远程执行脚本，同时请求控制台，如 `ssh user@remote -t 'echo $PWD'：non-login + interactive`
		* 在图形化界面中打开 `terminal`：
			* `Linux` 上: `non-login + interactive`
			* `Mac OS X` 上: `login + interactive`
	
* 回到当初的问题
	* 很显然我们遇到的问题是属于 `non-login  non-interactive` 模式，对于这种模式，bash 会选择加载 `$BASH_ENV` 的值所对应的文件，所以为了让它加载 `/etc/profile`，可以设定：

			export BASH_ENV=/etc/profile
	* 但是尽管设置了这些还是有可能会出错，如果 脚本文件开头是 `#! /bin/sh` 或者以 sh 结尾，这是因为如果 bash 以 sh 启动会使配置文件的加载顺序发生变化

			ll `which sh`
			lrwxrwxrwx 1 root root 9 Apr 25  2014 /usr/bin/sh -> /bin/bash
	* 如果bash 以 sh 启动配置文件的加载顺序如下：
		* `interactive + login`: 读取 `/etc/profile 和 ~/.profile`
		* `non-interactive + login`: 同上
		* `interactive + non-login`: 读取 `ENV` 环境变量对应的文件
		* `non-interactive + non-login`: 不读取任何文件
	* 这样便可以解释为什么出错了，因为这里属于 `non-interactive + non-login`，所以 `bash` 不会读取任何文件，故而即使设置了 `BASH_ENV` 也不会起作用。所以为了解决问题，只需要把 `sh` 换成 `bash`，再设置环境变量 `BASH_ENV` 即可，或者在第一行结尾加入 **`--login`** ，强制将 `bash` 转换成 `login shell`

* 配置文件的建议
	* 回顾一下前面提到的所以配置文件一共有如下：
		* `/etc/profile`
		* `~/.bash_profile`
		* `~/.bash_login`
		* `~/.profile`
		* `/etc/bash.bashrc`
		* `~/.bashrc`
		* `$BASH_ENV`
		* `$ENV`
	* 配置文件最佳实践供
		* `~/.bash_profile`：应该尽可能的简单，通常会在最后加载 `.profile` 和 `.bashrc` (注意顺序)
		* `~/.bash_login`：在前面讨论过，别用它
		* `~/.profile`：此文件用于 `login shell`，所有你想在整个用户会话期间都有效的内容都应该放置于此，比如启动进程，环境变量等
		* `~/.bashrc`：只放置与 `bash` 有关的命令，所有与交互有关的命令都应该出现在此，比如 bash 的补全、alias、颜色、提示符等等