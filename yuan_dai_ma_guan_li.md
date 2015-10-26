# 源代码管理工具
- CVS
- SVN:集中式源代码管理工具
	- 服务器管理所有的代理更新与下载
- GIT:分布式源代码管理工具
	- 服务器把任务分发下去：本地会有一个大的仓库，平时提交只需在本地，在可以联网时可以再提交到服务器
## SVN
- 基本操作
	- 服务器端
		- svn 仓库`Repository`
		- 安装：Windows-`Visual SVN Server`
	- 客户端
		- svn checkout 服务器地址 --username=用户名 --password=密码（下载到本地:只需要做一次）
		- svn commit -m "注释"(提交到服务器)
		- svn update （更新到本地）
	- 使用方式：先update 再commit
	- SVN基本命令
		- 命令行格式：
		- svn <subcommand> [options] [args]
		- 说明 svn 子命令 [选项] [参数]
- 使用
	- 1、先在服务器建立一个服务器库，指定用户或组，获得服务器地址
	- 2、建立本地库(用户名和密码可写可不写，但是都需要输入) 只需要执行一次
		- svn checkout http://192.168.82.130/svn/weixin
	- 3、创建测试文件
		- cd weixin/code // 进入代码目录
		- touch main.c // 创建并编辑文件
		- svn add main.c // 添加到本地库
		- svn commit -m "添加了main.c文件" // 更新到服务器
		- svn revert Person.h // 撤销修改
		- svn state // 查看工作目录状态
		- svn log // 查看日志
		- svn
	- 4、代码问题冲突
		- 两个人同时修改同一个文件,提交时就会出问题

```
		song$svn ci -m "添加了number22"
		Sending        Person.h
		svn: E160042: Commit failed (details follow):
		svn: E160042: File or directory 'Person.h' is out of date; try updating
		svn: E160024: resource out of date; try updating
		song$ svn update
		Updating '.':
		Conflict discovered in '/Users/song/Desktop/code/SVN/slq/weixin/code/Person.h'.
		Select: (p) postpone, (df) diff-full, (e) edit,
		        (mc) mine-conflict, (tc) theirs-conflict,
		        (s) show all options:
		// 常用选择
		(p) postpone            两个都要
		(mc) mine-conflict      使用我的
		(tc) theirs-conflict    使用对方的
```
- svn st 显示的文件状态
	-第1列状态说明：描述文件被添加、删除或其他修改

		' ' 没有修改
		'A' 被添加到本地代码仓库
		'C' 冲突
		'D' 被删除
		'I' 被忽略
		'M' 被修改
		'R' 被替换
		'X' 外部定义创建的版本目录
		'?' 文件没有被添加到本地版本库内
		'!' 文件丢失或者不完整（不是通过svn命令删除的文件）
		'~' 受控文件被其他文件阻隔

- xcode与svn集成
- 代码冲突
	- 本机版本低于服务器版本时就会产生代码冲突
	- 提交冲突：error：out of
	- 更新冲突：conflict
	- 回退（待解决）
	- 直接使用合并命令
		- svn merger -r 15:9 ViewController.m
- 图形化界面工具

	- 静态库-只能命令行添加
- 目录规范
	- 主干
	- 分支
	- 标记


## GIT
- GIT:分布式源代码管理工具
	- 服务器把任务分发下去：本地会有一个大的仓库，平时提交只需在本地，在可以联网时可以再提交到服务器
- 命令行代码
	- git init // 创建git仓库
	- git config user.name song // 配置用户信息
	- git config user.email song@163.com
	- git config -l // 查看配置信息
	- git status // 查看文件状态
	- git add main.m // 添加文件到缓存区
	- git commit -m "注释" // 添加文件到本地仓库
	- git add . // 添加当前目录下得所有文件到缓存区
	- git config alias.st status // 给命令status起别名
	- git config alias.ci "commit -m" // 给命令""commit -m"起别名
	- git log // 查看日志信息
	- 配置带颜色的log别名
		- git config alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
	- git reset --hard HEAD // 放弃当前所有修改，回到当前版本
	- git reset --hard HEAD^ // 回到上一个版本，^ 表示回退几个版本，^^表示回退两个版本
	- git reset --hard HEAD~10 // 回退10个版本
	- git reset --hard HEAD e7823ji // 回退到指定版本，后面写的是版本号
	- git reflog // 查看所有的版本信息，包括回退后的

- 多人开发
	- 本地共享仓库
	- git init --bare // 建立空白代码库
    - ---------------
	- git clone 路劲 // 建立自己的仓库
	- 添加.gitignore，忽略某些文件
	- git add .gitignore
	- git commit -m ""
	- 修改文件
	- git push origin master // 推送到本地仓库


## trunk


- 上传代码带github


- 上传文件到cocoapod
- 注册，要在邮箱里进行验证

`pod trunk register  邮箱 '用户名' --description='电脑描述'`

- 查看属性 `pod trunk me`

```objc
  - Name:     song
  - Email:    slq0378@163.com
  - Since:    August 2nd, 08:03
  - Pods:     None
  - Sessions:
    - August 2nd, 08:03 - December 8th, 08:46. IP: 113.111.199.171
    Description: songMac
```
- 这时候需要尝试更新gem源或者pod
	* `sudo gem update --system`
	* `sudo gem install cocoapods`
	* `sudo gem install cocospods-trunk`
- 接下来需要在项目根路径创建一个podspec文件来描述你的项目信息
	* `pod spec cretae test`
    * 创建一个test.podspec文件
    * 值得注意的是，现在的podspec必须有tag，所以最好先打个tag，传到github
    	* `git tag 0.0.1`
    	* `git push --tags`
    * 然后打开test.podspec 文件修改里面文件

```objc
Pod::Spec.new do |s|
  s.name     = 'SLQStatusBar'
  s.version  = '0.0.1'
  s.license  = 'MIT'
  s.summary  = "Showing message on the statusBar."
  s.homepage = 'https://github.com/slq0378'
  s.authors  = { 'Mr Song' =>
                 'slq0378@163.com' }
  s.social_media_url = "http://www.cnblogs.com/songliquan/"
  s.source   = { :git => 'https://github.com/slq0378/SLQTest.git', :tag => '0.0.1' }
  s.source_files = 'SLQStatusBar'
end
```
- 这个source_files目前还不会写

- 检测podspec语法
    * `pod spec lint test.podspec`


- 发布podspec
    * `pod trunk push test.podspec`
    * 如果是第一次发布pod，需要去https://trunk.cocoapods.org/claims/new认领pod

- 测试
    * `pod setup` : 初始化
    * `pod repo update` : 更新仓库
    * `pod search Test`



