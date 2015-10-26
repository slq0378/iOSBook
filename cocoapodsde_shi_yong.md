## 更换源
- Gem是一个管理Ruby库和程序的标准包，它通过Ruby Gem(如 http://rubygems.org/ )源来查找、安装、升级和卸载软件包
- gem sources --remove https://rubygems.org/
- gem sources -a http://ruby.taobao.org/
- gem sources -l

## 更新升级gem
- sudo gem update --system

## 安装
- sudo gem install cocoapods

## 更换repo镜像为国内服务器
- pod repo remove master
- pod repo add master https://gitcafe.com/akuandev/Specs.git
- pod repo add master http://git.oschina.net/akuandev/Specs.git(貌似不行)

## 初始化第三方库信息
- pod setup

## 以后更新第三方库信息
- pod repo update

## 搜索
- pod search

## 新建Podfile
- vim Podfile
- 输入i：进入编辑状态
- 输入dd：删除当前行
- 按ESC：退出编辑模式
- 输入:wq：保存并退出
- Podfile文件的格式

```objc
platform :ios, '8.0'
pod '框架名字'
pod '框架名字', '~> 版本号'
```

## 解析Podfile，安装第三方框架
- pod install

## 解析Podfile，升级第三方框架
- pod update

## 以后使用CocoaPods过程中出现了莫名其妙的问题
- sudo gem update --system
- sudo gem install cocoapods
- pod setup
