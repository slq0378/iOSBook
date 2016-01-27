# 证书打包发布

# 证书
## 一.App ID（bundle identifier）
- App ID即Product ID，用于标识一个或者一组App。
- App ID应该和Xcode中的Bundle Identifier是一致（Explicit）的或匹配（Wildcard）的。
- App ID字符串通常以反域名（reverse-domain-name）格式的Company Identifier（Company ID）作为前缀（Prefix/Seed），一般不超过255个ASCII字符。
- App ID全名会被追加Application Identifier Prefix（一般为TeamID.），分为两类：
- Explicit App ID：唯一的App ID，用于唯一标识一个应用程序。例如“com.apple.garageband”这个App ID，用于标识Bundle Identifier为“com.apple.garageband”的App。
- Wildcard App ID：含有通配符的App ID，用于标识一组应用程序。例如“*”（实际上是Application Identifier Prefix）表示所有应用程序；而“com.apple.*”可以表示Bundle Identifier以“com.apple.”开头（苹果公司）的所有应用程序。
- 用户可在Developer MemberCenter网站上注册（Register）或删除（Delete）已注册的App IDs。
- App ID被配置到【XcodeTarget|Info|Bundle Identifier】下；对于Wildcard App ID，只要bundle identifier包含其作为Prefix/Seed即可。

## 二.设备（Device）
- Device就是运行iOS系统用于开发调试App的设备。每台Apple设备使用UDID来唯一标识。
- iOS设备连接Mac后，可通过iTunes->Summary或者Xcode->Window->Devices获取iPhone的UDID（identifier）。
- Apple Member Center网站个人账号下的Devices中包含了注册过的所有可用于开发和测试的设备，普通个人开发账号每年累计最多只能注册100个设备。
	- Apps signed by you or your team run only on designated development devices.
	- Apps run only on the test devices you specify.
- 用户可在网站上注册或启用/禁用（Enable/Disable）已注册的Device。
- 本文的Devices是指连接到Xcode被授权用于开发测试的iOS设备（iPhone/iPad）。

## 三.开发证书（Certificates）
- 1.证书的概念
	- 证书是由公证处或认证机关开具的证明资格或权力的证件，它是表明（或帮助断定）事理的一个凭证。证件或凭证的尾部通常会烙印公章。
	- 每个中国人一生可能需要70多个证件，含15种身份证明。证件中“必需的”有30到40个。将这些证件按时间顺序铺开，那就是一个天朝子民的一生——持准生证许可落地，以户籍证明入籍，以身份证认证身份，持结婚证以合法同居，最终以死亡证明注销。
- 2.数字证书的概念
	- 数字证书就是互联网通讯中标志通讯各方身份信息的一串数字，提供了一种在Internet上验证通信实体身份的方式，其作用类似于司机的驾驶执照或日常生活中的身份证。它是由一个由权威机构——CA机构，又称为证书授权中心（Certificate Authority）发行的，人们可以在网上用它来识别对方的身份。
	- 数字证书是一个经证书授权中心数字签名的包含公开密钥拥有者信息以及公开密钥的文件。最简单的证书包含一个公开密钥、名称以及证书授权中心的数字签名。
	- 数字证书还有一个重要的特征就是时效性：只在特定的时间段内有效。
- 数字证书中的公开密钥（公钥）相当于公章。
- 某一认证领域内的根证书是CA认证中心给自己颁发的证书，是信任链的起始点。安装根证书意味着对这个CA认证中心的信任。
- 为了防止GFW进行中间人攻击(MitM)，例如篡改github证书，导致无法访问github网站等问题，可选择不信任CNNIC
- 3.iOS（开发）证书
	- iOS证书是用来证明iOS App内容（executable code）的合法性和完整性的数字证书。对于想安装到真机或发布到AppStore的应用程序（App），只有经过签名验证（Signature Validated）才能确保来源可信，并且保证App内容是完整、未经篡改的。
	- iOS证书分为两类：Development和Production（Distribution）。
		- Development证书用来开发和调试应用程序：A development certificate identifies you, as a team member, in a development provisioning profile that allows apps signed by you to launch on devices. 
		- Production主要用来分发应用程序（根据证书种类有不同作用）：A distribution certificate identifies your team or organization in a distribution provisioning profile and allows you to submit  your app to the store. Only a team agent or an admin can create a distribution certificate.
	- 普通个人开发账号最多可注册iOS Development/Distribution证书各2个，用户可在网站上删除（Revoke）已注册的Certificate。
	- 下文主要针对iOS App开发调试过程中的开发证书（Certificate for Development）。
- 4.iOS（开发）证书的根证书
	- 那么，iOS开发证书是谁颁发的呢？或者说我们是从哪个CA申请到用于Xcode开发调试App的证书呢？
iOS以及Mac OS X系统（在安装Xcode时）将自动安装AppleWWDRCA.cer这个中间证书（Intermediate Certificates），它实际上就是iOS（开发）证书的证书，即根证书（Apple Root Certificate）。
	- AppleWWDRCA（Apple Root CA）类似注册管理户籍的公安机关户政管理机构，AppleWWDRCA.cer之于iOS（开发）证书则好比户籍证之于身份证。
	- 如果Mac Keychain Access证书助理在申请证书时尚未安装过该证书，请先下载安装（Signing requires that you have both the signing identity and the intermediate certificate installed in your keychain）。
- 5.申请证书（CSR：Certificate Signing Request）
	- 可以在缺少证书时通过Xcode Fix Issue自动请求证书，这里通过Keychain证书助理从证书颁发机构请求证书：填写开发账号邮件和常用名称，勾选【存储到磁盘】。
	- keychain将生成一个包含开发者身份信息的CSR（Certificate Signing Request）文件；同时，Keychain Access|Keys中将新增一对Public/Private Key Pair（This signing identity consists of a public-private key pair that Apple issues）。
	- private key始终保存在Mac OS的Keychain Access中，用于签名（CodeSign）对外发布的App；public key一般随证书（随Provisioning Profile，随App）散布出去，对App签名进行校验认证。用户必须保护好本地Keychain中的private key，以防伪冒。
		- Keep a secure backup of your public-private key pair. If the private key is lost, you’ll have to create an entirely new identity to sign code. 
		- Worse, if someone else has your private key, that person may be able to impersonate you.
	- 在Apple开发网站上传该CSR文件来添加证书（Upload CSR file to generate your certificate）：
	- Apple证书颁发机构WWDRCA(Apple Worldwide Developer Relations Certification Authority)将使用private key对CSR中的public key和一些身份信息进行加密签名生成数字证书（ios_development.cer）并记录在案（Apple Member Center）。
	
	- 从Apple Member Center网站下载证书到Mac上双击即可安装（当然也可在Xcode中添加开发账号自动同步证书和[生成]配置文件）。证书安装成功后，在KeychainAccess|Keys中展开创建CSR时生成的Key Pair中的私钥前面的箭头，可以查看到包含其对应公钥的证书（Your requested certificate will be the public half of the key pair.）；在Keychain Access|Certificates中展开安装的证书（ios_development.cer）前面的箭头，可以看到其对应的私钥。
	
	- Certificate被配置到【Xcode Target|Build Settings|Code Signing|Code Signing Identity】下，下拉选择Identities from Profile "..."（一般先配置Provisioning Profile）。
	

## 四.供应配置文件（Provisioning Profiles）
- 1.Provisioning Profile的概念
	- 一个Provisioning Profile对应一个Explicit App ID或Wildcard App ID（一组相同Prefix/Seed的App IDs）。在网站上手动创建一个Provisioning Profile时，需要依次指定App ID（单选）、证书（Certificates，可多选）和设备（Devices，可多选）。用户可在网站上删除（Delete）已注册的Provisioning Profiles。
	- Provisioning Profile决定Xcode用哪个证书（公钥）/私钥组合（Key Pair/Signing Identity）来签署应用程序（Signing Product）,将在应用程序打包时嵌入到.ipa包里。安装应用程序时，Provisioning Profile文件被拷贝到iOS设备中，运行该iOS App的设备也通过它来认证安装的程序。
	- 如果要打包或者在真机上运行一个APP，一般要经历以下三步：
		- 首先，需要指明它的App ID，并且验证Bundle ID是否与其一致；
		- 其次，需要证书对应的私钥来进行签名，用于标识这个APP是合法、安全、完整的；
		- 然后，如果是真机调试，需要确认这台设备是否授权运行该APP。
	- Provisioning Profile把这些信息全部打包在一起，方便我们在调试和发布程序打包时使用。这样，只要在不同的情况下选择不同的Provisioning Profile文件就可以了。
	- Provisioning Profile也分为Development和Distribution两类，有效期同Certificate一样。Distribution版本的ProvisioningProfile主要用于提交App Store审核，其中不指定开发测试的Devices（0，unlimited）。App ID为Wildcard App ID（*）。App Store审核通过上架后，允许所有iOS设备（Deployment Target）上安装运行该App。
	- Xcode将全部供应配置文件（包括用户手动下载安装的和Xcode自动创建的Team Provisioning Profile）放在目录~/Library/MobileDevice/Provisioning Profiles下。

## 五.开发组供应配置文件（Team Provisioning Profiles）
- 1.Team Provisioning Profile的概念
	- 每个Apple开发者账号都对应一个唯一的Team ID，Xcode3.2.3预发布版本中加入了Team Provisioning Profile这项新功能。
	- 在Xcode中添加Apple Developer Account时，它将与Apple Member Center后台勾兑自动生成iOS Team Provisioning Profile（Managed by Xcode）。
	- Team Provisioning Profile包含一个为Xcode iOS Wildcard App ID(*)生成的iOS Team Provisioning Profile:*（匹配所有应用程序），账户里所有的Development Certificates和Devices都可以使用它在这个team注册的所有设备上调试所有的应用程序（不管bundle identifier是什么）。同时，它还会为开发者自己创建的Wildcard/Explicit App IDs创建对应的iOS Team Provisioning Profile。
- 2.Team Provisioning Profile生成/更新时机
	- Add an Apple ID account to Xcode
	- Fix issue "No Provisioning Profiles with a valid signing identity" in Xcode
	- Assign Your App to a Team in Xcode project settings of General|Identity
	- Register new device on the apple development website or Xcode detected new device connected
	- 利用Xcode生成和管理的iOS Team Provisioning Profile来进行开发非常方便，可以不需要上网站手动生成下载Provisioning Profile。
	- Team Provisioning Profile同Provisioning Profile，只不过是由Xcode自动生成的，也被配置到【XcodeTarget|Build Settings|Code Signing|Provisioning Profile】下。

## 六.App Group （ID）
- 1.App Group的概念
	- WWDC14除了发布了OS X v10.10和switf外，iOS 8.0也开始变得更加开放了。说到开放，当然要数应用扩展（App Extension）了。顾名思义，应用扩展允许开发者扩展应用的自定义功能和内容，能够让用户在使用其他应用程序时使用该项功能，从而实现各个应用程序间的功能和资源共享。可以将扩展理解为一个轻量级（nimble and lightweight）的分身。
	- 扩展和其Containing App各自拥有自己的沙盒，虽然扩展以插件形式内嵌在Containing App中，但是它们是独立的二进制包，不可以互访彼此的沙盒。为了实现Containing App与扩展的数据共享，苹果在iOS 8中引入了一个新的概念——App Group，它主要用于同一Group下的APP实现数据共享，具体来说是通过以App Group ID标识的共享资源区——App Group Container。
	- App Group ID同App ID一样，一般不超过255个ASCII字符。用户可在网站上编辑Explicit App IDs的App Group Assignment；可以删除（Delete）已注册的AppGroup （ID）。
- 2.App Group的配置
	- Containing App与Extension的Explicit App ID必须Assign到同一App Group下才能实现数据共享，并且Containing App与Extension的App ID命名必须符合规范：
	- 置于同一App Group下的App IDs必须是唯一的（Explicit，not Wildcard）
	- Extension App ID以Containing App ID为Prefix/Seed
	- 假如Garageband这个App ID为“com.apple.garageband”，则支持从语音备忘录导入到Garageband应用的插件的App ID可能形如“com.apple.garageband.extImportRecording”。

	- 关于Provisioning Profile，可以使用自己手动生成的，也可以使用Xcode自动生成的Team Provisioning Profile。
	- App Group会被配置到【Xcode Target|Build Settings|Code Signing|Code Signing Entitlements】文件（*.entitlements）的键com.apple.security.application-groups下，不影响Provisioning Profile生成流程。

## 七.证书与签名（Certificate& Signature）
- 1.Code Signing Identity
	- Xcode中配置的Code Signing Identity（entitlements、certificate）必须与Provisioning Profile匹配，并且配置的Certificate必须在本机Keychain Access中存在对应Public／Private Key Pair，否则编译会报错。
	- Xcode所在的Mac设备（系统）使用CA证书（WWDRCA.cer）来判断Code Signing Identity中Certificate的合法性：
	- 若用WWDRCA公钥能成功解密出证书并得到公钥（Public Key）和内容摘要（Signature），证明此证书确乃AppleWWDRCA发布，即证书来源可信；
	- 再对证书本身使用哈希算法计算摘要，若与上一步得到的摘要一致，则证明此证书未被篡改过，即证书完整。
	
- 2.Code Signing
	- 每个证书（其实是公钥）对应Key Pair中的私钥会被用来对内容（executable code，resources such as images and nib files aren’t signed）进行数字签名（CodeSign）——使用哈希算法生成内容摘要（digest）。
	- Xcode使用指定证书配套的私钥进行签名时需要授权，选择【始终允许】后，以后使用该私钥进行签名便不会再弹出授权确认窗口。
	
	