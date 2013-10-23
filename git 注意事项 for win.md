#### Git 在 windows 下使用会有编码的问题，以下是一些注意事项和配置：

	使用 git 时，如果是跨平台的项目，在 win 下设置 core.autocrlf = true, 在 mac 和 linux 下设置 core.autocrlf = input，这样可以保证换行符不会出问题

#### windows 下的 git bash 用 ls 命令时中文名文件显示乱码

	%GIT_HOME%\etc\git-completion.bash
 
	alias ls='ls --show-control-chars --color=auto'

	alias ll='ls -l'

#### 使用 git add添加要提交的文件的时候，如果文件名是中文，会显示形如`274\232\...\273\223.png`的乱码。

	git config --global core.quotepath false

#### 使用git log显示提交的中文log乱码。

	git config --global gui.encoding utf-8

#### 设置 commit log 提交时使用 utf-8 编码，可避免服务器上乱码，同时与 linux上的提交保持一致！

	git config --global i18n.commitencoding utf-8

#### 使得在 $ git log 时将 utf-8 编码转换成 gbk 编码，解决 Msys bash 中 git log 乱码。

	git config --global i18n.logoutputencoding gbk