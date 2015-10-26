# 网络

## 基本概念

- 客户端：client
- 服务器：server
- 请求:request
- 响应:response
- 过程
	- 客户端 -> 发送请求 -> 服务器(连接数据库)
	- 服务器 -> 发送响应 -> 客户端
- 客户端(移动端)
	- 前段（前台）
	- iOS,Android
- 服务器(后端)
	- 后台
	- Java、PHP、.NET
	- 远程服务器-面向所有用户（上线）
	- 本地服务器-面向公司内部（测试）
- URL
	- URL的全称是Uniform Resource Locator（统一资源定位符）
	- 通过1个URL，能找到互联网上唯一的1个资源
	- URL就是资源的地址、位置，互联网上的每个资源都有一个唯一的URL
	- URL的基本格式 = 协议://主机地址/路径
- URL中常见协议
	- **HTTP** - 网络所有资源（http://）这个最常用
	- file - 本机资源（file://）
	- mailto - 邮件地址 (mailto:)
	- FTP - 网络文件资源 （ftp://）

## HTTP通信
- http的作用是统一客户端和服务器的数据交换格式，使得彼此可以理解。
- 优点
	- 简单快速，服务器规模小，通信速度快
	- 灵活，允许传输任意类型的数据
	- HTTP 0.9和1.0使用短连接方式（非持续连接）：每次连接只处理一个请求，服务器做出响应后，马上断开连接。

## iOS中常用的http
- 原生
	- `NSURLConnection` - 最古老的方案
	- `NSURLSession` - iOS7推出的新技术
	- `CFNetworking` - NSURL的底层，纯C语言
- 第三方框架
	- `ASIHttpRequest` - 外号“HTTP终结者”，功能强大，可惜已经停止更新
	- `AFNetworking` - 维护使用者多
	- `MKNetworkKit` - 印度，维护使用者少
- HTTP 请求
	- HTTP/1.1 中定义了8种方法
	- **GET**、**POST**、OPTIONS、HEAD、PUT、DELETE、TRACE、CONNECT、PATCH
	- 最常用的就是GET、POST

## HTTP 实践 - `NSURLConnection`(了解即可)
- HTTP同步请求

```objc
// 发送同步请求，会一直等待， 直到接收到数据
- (void)requestSynch
{
    // 1 创建请求连接
    NSURL *url = [NSURL URLWithString:@"http://www.lala.com/login?username=123&pwd=123"];
    // 2 创建请求
    NSURLRequest *request = [NSURLRequest requestWithURL:url ];

    NSHTTPURLResponse *response = nil;
    NSError *error = nil;
    // 3 发送同步请求
    // endSynchronousRequest阻塞式的方法，等待服务器返回数据
    NSData *data = [NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&error];

    // 4.解析服务器返回的数据（解析成字符串）
    NSString *str = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

    NSLog(@"--%@--%@-- %zd",str,response.allHeaderFields,response.statusCode);

}
```
- HTTP异步请求

```objc
// 发送异步请求
- (void)requestAsync
{
    // 1 创建请求连接
    NSURL *url = [NSURL URLWithString:@"http://123.123.123.123/login?username=123&pwd=123"];
    // 2 创建请求
    NSURLRequest *request = [NSURLRequest requestWithURL:url ];
    // 3 发送异步请求
    [NSURLConnection sendAsynchronousRequest:request queue:[[NSOperationQueue alloc] init] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
        // 4.解析服务器返回的数据（解析成字符串）
        NSString *str = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
        NSHTTPURLResponse *httpRes = (NSHTTPURLResponse* )response;
        NSLog(@"--%@--%zd--%@--",str,httpRes.statusCode,httpRes.allHeaderFields);
    }];
}
```

- http代理请求模式
	- 协议：NSURLConnectionDataDelegate
	- 实现方法

```objc
    // 1 创建请求连接
    NSURL *url = [NSURL URLWithString:@"http://123.123.123.123/login?username=520it&pwd=520it"];
    // 2 创建请求
    NSURLRequest *request = [NSURLRequest requestWithURL:url ];

    // 3 代理请求模式,要遵守协议并实现代理方法
    [NSURLConnection connectionWithRequest:request delegate:self];

///-----------------------------------------------------------------///

// 常用代理方法
// 接收服务器响应
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
{
    self.localData = [NSMutableData data];
     NSLog(@"-didReceiveResponse-%zd",((NSHTTPURLResponse *)response).statusCode);
}
// 接收到服务器的数据（如果数据量比较大，这个方法会被调用多次）
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{
    [self.localData appendData:data];
      NSLog(@"-didReceiveData-");
}
// 完成数据下载
- (void)connectionDidFinishLoading:(NSURLConnection *)connection
{
    NSString *str = [[NSString alloc] initWithData:self.localData encoding:NSUTF8StringEncoding];

    NSLog(@"-connectionDidFinishLoading-%@",str);
}
// 请求失败：请求超时等
- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error
{
     NSLog(@"-didFailWithError-");
}
```

## 简单登陆界面

- GET请求

```objc
- (IBAction)loginBtn:(id)sender {

    // 获取控件数据和网络数据进行比对
    NSString *userName = self.userText.text;
    if (userName.length == 0 ) {
        [SVProgressHUD showErrorWithStatus:@"用户名不能为空"];
        return;
    }

    NSString *pwd = self.pwdText.text;
    if (pwd.length == 0) {
        [SVProgressHUD showErrorWithStatus:@"密码不能为空"];
        return;
    }

    // 显示阴影
    [SVProgressHUD showWithStatus:@"正在登陆中" maskType:SVProgressHUDMaskTypeBlack];
    //
    NSString *format = [NSString stringWithFormat:@"http://123.123.123.123/login?username=%@&pwd=%@",userName,pwd];
    NSLog(@"%@",format);
    // 1 创建请求连接
    NSURL *url = [NSURL URLWithString:format];
    // 2 创建请求
    NSURLRequest *request = [NSURLRequest requestWithURL:url ];

    // 3 发送异步请求
    [NSURLConnection sendAsynchronousRequest:request queue:[[NSOperationQueue alloc] init] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {

        // JSON:{"success":"登录成功"}
        // 4.解析服务器返回的数据（解析成字符串）
        NSString *str = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
        // 对字符串进行分割
        NSInteger loc = [str rangeOfString:@"\":\""].location + 3;
        NSInteger len = [str rangeOfString:@"\"}"].location - loc;
        NSString *result = [str substringWithRange:NSMakeRange(loc, len)];
        // 显示结果
        if ([result containsString:@"success"]) {
            [SVProgressHUD showSuccessWithStatus:result];
        }
        else
        {
            [SVProgressHUD showErrorWithStatus:result];
        }
    }];
```

- POST请求

```objc
   NSString *format = [NSString stringWithFormat:@"http://123.123.123.123/login"];

    // 1 创建请求连接
    NSURL *url = [NSURL URLWithString:format];
    // 2 创建请求
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url ];

    // 更改请求方法
    request.HTTPMethod = @"POST";
    // 设置请求体
    request.HTTPBody = [@"username=520it&pwd=520it" dataUsingEncoding:NSUTF8StringEncoding];
    // 设置超时
    request.timeoutInterval = 5;
    // 设置请求头
//    [request setValue:@"ios9.0" forHTTPHeaderField:@"User-agent"];
    // 3 发送请求
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
        if (connectionError) {
            NSLog(@"失败");
        }
        else
        {
            NSLog(@"%@",[[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
        }
    }];
```
- **路径中有汉字的话必须进行转换**
- GET:手动转换

```objc
    format = [format stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
```
- POST:在设置请求体时直接设置了编码格式，会自动转换

```objc
// 设置请求体
    request.HTTPBody = [@"username=小码哥&pwd=520it" dataUsingEncoding:NSUTF8StringEncoding];
```
## 解析JSON
- 服务器返回给客户端的数据一般都是JSON或者XML
- JSON的格式很像OC中的字典和数组
	- {"name" : "jack", "age" : 10}
	- {"names" : ["jack", "rose", "jim"]}
- 标准JSON格式的注意点：key必须用双引号
- JSON和OC的对应关系

JSON  |	 OC
------| ------
{ }	|	字典 NSDictionry
[]	|	数组 NSArray
“ ”	| 	字符串 NSString
10,10.4	|  NSNumber

### IOS中JSON解决方案

- 第三方
	- JSONKit、SBJson、TouchJSON（性能从左到右，越差）
- 苹果原生（自带）：NSJSONSerialization（性能最好）

### NSJSONSerialization

- 解析JSON
- data转JSON

```objc
+ (id)JSONObjectWithData:(NSData *)data options:(NSJSONReadingOptions)opt error:(NSError **)error;

  // data转JSON
        NSString *str = [NSJSONSerialization JSONObjectWithData:data options:kNilOptions
```
- JSON转data

```objc
+ (NSData *)dataWithJSONObject:(id)obj options:(NSJSONWritingOptions)opt error:(NSError **)error;
```
- 关于参数

```objc
typedef NS_OPTIONS(NSUInteger, NSJSONReadingOptions) {
    NSJSONReadingMutableContainers = (1UL << 0), // 可变容器
    NSJSONReadingMutableLeaves = (1UL << 1), // 子节点也是可变的，也就是说转换的所有数据都是可变的
    NSJSONReadingAllowFragments = (1UL << 2) // 接收零散数据，比如说单个的‘10’，'false'
} NS_ENUM_AVAILABLE(10_7, 5_0);
```
- 参数`NSJSONReadingAllowFragments`使用如下

```objc
    // 参数NSJSONReadingAllowFragments 用来读取单个元素
    NSString *str  = @"10";
    NSData *data = [NSJSONSerialization JSONObjectWithData:[str dataUsingEncoding:NSUTF8StringEncoding] options:NSJSONReadingAllowFragments error:nil];
    NSLog(@"-- %@ --",data);
```


- JSON转模型
- 每次都手动去转换的话，非常费时费力。可以使用第三方框架
- MJExtension

```objc
 #import <MJExtension.h>
// 获得视频的模型数组
self.videos = [SLQVideo objectArrayWithKeyValuesArray:dict[@"videos"]];

```
## 苹果自带的movie播放器
- 只能播放有限的几种格式（mp4、mov等）

```objc
   // 播放视频
    NSURL *url = [NSURL URLWithString:@"http://123.123.123.123"];
    // MPMoviePlayerViewController // 视图控制器
    // MPMoviePlayerController // 内部没有view，不能直接弹出
    MPMoviePlayerViewController *vc = [[MPMoviePlayerViewController alloc] initWithContentURL:[url URLByAppendingPathComponent:video[@"url"]]];
    // modal 出视频并播放
     [self presentViewController:vc animated:YES completion:nil];
```

## 字典转模型框架
- Mantle
    - 所有模型都必须继承自MTModel
- JSONModel
    - 所有模型都必须继承自JSONModel
- MJExtension
    - 不需要强制继承任何其他类


## 设计框架需要考虑的问题
- 侵入性
    - 侵入性大就意味着很难离开这个框架
- 易用性
    - 比如少量代码实现N多功能
- 扩展性
    - 很容易给这个框架增加新框架

## 解析XML

- XML解析方式
	- SAX：逐行解释
	- DOM：一次性加载到内存
- 苹果自带：NSXMLParser
	- SAX 方式
- 第三方库
	- libxml2 ： 纯C语言，默认包含在iOS SDK  中，，同时支持SAX、DOM
	- GDataXML : DOM方式，由google开发，基于libxml2
- 建议
	- 大文件 ： NSXMLParser、libxml2
	- 小文件 ： GDataXML、XSXMLParser、libxml2

### XSXMLParser
- 使用过程
	- 1、设置源，初始化一个XSXMLParser对象
	- 2、设置代理，实现代理方法
	- 3、启动扫描

```objc
// 初始化方法
- (instancetype)initWithContentsOfURL:(NSURL *)url;  // initializes the parser with the specified URL.
- (instancetype)initWithData:(NSData *)data; // create the parser from data
```
- 使用方法

```objc
// 1 使用苹果的XML类解析xml文档
NSXMLParser *parser = [[NSXMLParser alloc] initWithData:data];
parser.delegate = self; // 2 设置代理
[parser parse]; // 3 启动扫描
```
- 主要是代理方法的使用

```objc
 #pragma mark -NSXMLParserDelegate代理方法

// 读取文档开始
- (void)parserDidStartDocument:(NSXMLParser *)parser
{
//    NSLog(@"parserDidStartDocument");
//    self.videoArray = [NSMutableArray array];
}

// 读取文档结束
-  (void)parserDidEndDocument:(NSXMLParser *)parser
{
//    NSLog(@"parserDidEndDocument--%@",self.videoArray);


}

// 开始读取元素，attributes 参数表示要读取的元素
- (void)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName attributes:(NSDictionary *)attributeDict{
    //    NSLog(@"didStartElement---%@",attributeDict);

    // 所有的元素都会经过这里进行读取,包括根
    // 所以要进行判断，把根给排除
    if ([elementName isEqualToString:@"videos"]) {
        return;
    }
    // 获取模型数据，放入数组
    SLQVideoItem *video = [SLQVideoItem objectWithKeyValues:attributeDict];
    [self.videoArray addObject:video];

}
// 读取元素结束
- (void)parser:(NSXMLParser *)parser didEndElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName
{

}
```

## 格式化服务器返回的JSON数据
- 在线格式化：http://tool.oschina.net/codeformat/json
- 将服务器返回的字典或者数组写成plist文件

### GDataXML
- 这个配置起来很麻烦
	- 首先不支持CocoaPods，所以只能手动拖拽进项目，而且项目比较旧。
	- 然后进入工程设置 `Header Search Paths` 添加` /usr/include/libxml2`
	- 设置工程 `Other Linker Flags contain` 添加 `-lxml2`
	- 改变文件的编译方式为非ARC - `-fno -objc -arc`
	- 最后编译才不会出错
- 使用过程
	- GDataXMLDocument - 整个文档
	- GDataXMLElement - xml中某个元素

```objc
        // 1 使用GDataXML解析xml文件
        GDataXMLDocument *doc = [[GDataXMLDocument alloc] initWithData:data options:0 error:nil];
        // 2 获取所有video元素，首先要获得根节点
        NSArray *elements = [doc.rootElement elementsForName:@"video"];
        // 3 遍历所有元素
        for (GDataXMLElement *ele in elements) {
            SLQVideoItem *video  = [[SLQVideoItem alloc] init];
            // 4 获取元素属性，转换成字符串
            video.name = [[ele attributeForName:@"name"] stringValue];
            video.image  = [[ele attributeForName:@"image"] stringValue];
            video.url = [[ele attributeForName:@"url"] stringValue];
            video.length = [[[ele attributeForName:@"length"] stringValue] intValue];
            [self.videoArray addObject:video];
        }
```

### 多值参数
- 同一个参数传如多个数据，逗号分隔
- 一个参数包括多个字典

```objc
// NSURL *url = [NSURL URLWithString:@"http://123.123.123.123/weather?place=beijing&place=shanghai"];
    NSURL *url = [NSURL URLWithString:@"http://123.123.123.123/weather?place=beijing,shanghai"];
```


### 解决输出到控制台显示中文乱码的问题

- 重写descriptionWithLocale方法,这二个方法会在转换JSON时自动调用。

```objc
@implementation NSDictionary (Log)

- (NSString *)descriptionWithLocale:(id)locale
{

    // {"weathers":[{"city":"beijing,shanghai","status":"晴转多云"}]}\

    // 将这句话格式化输出
    NSMutableString *str =[NSMutableString string];
    [str appendString:@"{\n"];

    [self enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
        // 拼接字符串
        [str appendFormat:@"\t%@",key];
        [str appendString:@" : "];
        [str appendFormat:@"%@,\n",obj];

    }];
    [str appendString:@"}"];

    // 取出最后一个逗号
    NSRange range = [str rangeOfString:@"," options:NSBackwardsSearch];
    if (range.location != NSNotFound) {

        [str deleteCharactersInRange:range];
    }

    return str;
}

@end
/*
 2015-07-15 18:43:08.137 05-掌握-多值参数[65936:116425]
 {
	weathers : [
	{
	status : 晴转多云,
	city : Beijing
 },
	{
	status : 晴转多云,
	city : Shanghai
 }
 ]
 }

 */
@implementation NSArray (Log)
- (NSString *)descriptionWithLocale:(id)locale
{
    // 将这句话格式化输出
    NSMutableString *str =[NSMutableString string];
    [str appendString:@"[\n"];
    [self enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
        [str appendFormat:@"\t%@,\n",obj];
    }];
    [str appendString:@"\t]"];

    // 取出最后一个逗号
    NSRange range = [str rangeOfString:@"," options:NSBackwardsSearch];
    if (range.location != NSNotFound) {

        [str deleteCharactersInRange:range];
    }
    return str;
}
@end
```

## 文件下载

### 小文件下载
- 直接使用NSData下载

	`NSData *data = [NSData dataWithContentsOfURL:url];`
- 使用NSURLConnection 异步连接
```objc
    NSURL *url  = [NSURL URLWithString:@"http://123.123.123.123/resources/../images/minion_13.png"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
        NSLog(@"---%zd---",data.length);
    }];
```
- 使用NSURLConnection代理方式实现

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    // 建立连接
    NSURL *url  = [NSURL URLWithString:@"http://123.123.123.123/resources/videos/minion_02.mp4"];
    [NSURLConnection connectionWithRequest:[NSURLRequest requestWithURL:url] delegate:self];
}

- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSHTTPURLResponse *)response
{
    NSLog(@"didReceiveResponse:%@",response);

    // 初始化
    self.data = [NSMutableData data];
    self.movieCount = [response.allHeaderFields[@"Content-Length"] integerValue];
}

- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{
    // 缓存到内存
    [self.data appendData:data];

    CGFloat progress = 1.0 * self.data.length / self.movieCount;
    self.progressView.progress = progress;
    NSLog(@"%f",progress * 100 );
}

- (void)connectionDidFinishLoading:(NSURLConnection *)connection
{
    // 这里进行文件的保存,保存到cache里

    NSString *filePath = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];

    [self.data writeToFile:[filePath stringByAppendingPathComponent:@"1.mp4"] atomically:YES];
    self.data = nil;
}
```
### 大文件下载
- 使用NSURLConnection代理方式实现

```objc
// 接收到响应的时候：创建一个空的文件
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSHTTPURLResponse *)response
{
    // 获取文件长度
    self.movieCount = [response.allHeaderFields[@"Content-Length"] integerValue];
    // 创建一个空文件
    [[NSFileManager defaultManager] createFileAtPath:SLQFilePath contents:nil attributes:nil];
    // 创建文件句柄
    self.handle = [NSFileHandle fileHandleForWritingAtPath:SLQFilePath];
 }
// 接收到具体数据：马上把数据写入一开始创建好的文件
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{
    // 移动到文件末尾
    [self.handle seekToEndOfFile];
    // 写入数据
    [self.handle writeData:data];
    //
    self.currentCount += data.length;
    CGFloat progress = 1.0 * self.currentCount / self.movieCount;
    self.progressView.progress = progress;
    NSLog(@"%f",progress * 100 );
}
- (void)connectionDidFinishLoading:(NSURLConnection *)connection
{
    self.movieCount = 0;
    // 关闭文件
    [self.handle closeFile];
    self.handle = nil;
}
```

## 解压缩
- 解压缩使用第三方库`SSZipArchive`

```objc
    // 1 使用指定文件，创建一个压缩文件
    NSArray *paths = @[
                       @"/Users/song/Desktop/test/1.png",
                       @"/Users/song/Desktop/test/2.png",
                       @"/Users/song/Desktop/test/3.png",
                       @"/Users/song/Desktop/test/4.png",
                       @"/Users/song/Desktop/test/5.png"
                       ];
    [Main createZipFileAtPath:@"/Users/song/Desktop/test.zip" withFilesAtPaths:paths];

	// 2 使用指定目录创建一个压缩文件
    [Main createZipFileAtPath:@"/Users/song/Desktop/test121212.zip" withContentsOfDirectory:@"/Users/song/Desktop/test"];
    // 3 解压缩一个文件到指定目录
    [Main unzipFileAtPath:@"/Users/song/Desktop/test.zip" toDestination:@"/Users/song/Desktop"];
```

## 文件上传

![](../images/MIMEType.png)



```objc
    // 一定要注意这个格式是固定的
    /* 文件参数格式
     --分割线\r\n
     Content-Disposition: form-data; name="参数名"; filename="文件名"\r\n
     Content-Type: 文件的MIMEType\r\n
     \r\n
     文件数据
     \r\n
     */
    // 文件上传
    // 1、创建请求
    NSURL *url = [NSURL URLWithString:@"http://123.123.123.123/upload"];

    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    // 2、设置协议
    request.HTTPMethod = @"POST";
    // 3、设置请求头
    [request setValue:[NSString stringWithFormat:@"multipart/form-data; boundary=%@",SLQBoundary] forHTTPHeaderField:@"Content-Type"];

    // 4、设置请求体
    NSMutableData *body = [NSMutableData data];

    // 设置文件参数
    // 设置边界
    [body appendData:SLQUTF(@"--")];
    [body appendData:SLQUTF(SLQBoundary)];
    [body appendData:SLQEnter];
    // 文件参数名
    [body appendData:SLQUTF([NSString[] stringWithFormat:@"Content-Disposition: form-data; name=\"file\"; filename=\"1.png\""])];
    [body appendData:SLQEnter];
    // 文件类型
    [body appendData:SLQUTF([NSString stringWithFormat:@"Content-Type: image/png"])];
    [body appendData:SLQEnter];
    // 文件内容
    [body appendData:SLQEnter];
    UIImage *image = [UIImage imageNamed:@"1"];
    [body appendData:UIImagePNGRepresentation(image)];
    [body appendData:SLQEnter];

    /* 非文件参数格式
     --分割线\r\n
     Content-Disposition: form-data; name="参数名"\r\n
     \r\n
     参数值
     \r\n
     */
    // 设置非文件参数
    [body appendData:SLQUTF(@"--")];
    [body appendData:SLQUTF(SLQBoundary)];
    [body appendData:SLQEnter];

    [body appendData:SLQUTF([NSString stringWithFormat:@"Content-Disposition: form-data; name=\"username\""])];
    [body appendData:SLQEnter];

    [body appendData:SLQEnter];
    [body appendData:SLQUTF(@"bulabulabula")];
    [body appendData:SLQEnter];

    /* 结束标记
     --分割--线\r\n
     \r\n
     */
    // 结束标记
    [body appendData:SLQUTF(@"--")];
    [body appendData:SLQUTF(SLQBoundary)];
    [body appendData:SLQUTF(@"--")];
    [body appendData:SLQEnter];


    request.HTTPBody = body;


    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
        //  这个方法会调用descriptionWithLocale方法，可以在这里解决输出到控制台显示中文乱码的问题
        NSLog(@"%@",[NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil]);
    }];
```

### 获取MIMEType

- OC 方法

```objc
- (NSString *)getMIMEType:(NSString *)path
{
    NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL fileURLWithPath:path]];
    NSURLResponse *response = nil;
    [NSURLConnection sendSynchronousRequest:request returningResponse:&response error:nil];
        return response.MIMEType;
}
```
- c语言

```objc
// 要包含头文件MobileCoreServices.h
+ (NSString *)mimeTypeForFileAtPath:(NSString *)path
{
    if (![[NSFileManager defaultManager] fileExistsAtPath:path]) {
        return nil;
    }
    CFStringRef UTI = UTTypeCreatePreferredIdentifierForTag(kUTTagClassFilenameExtension, (__bridge CFStringRef)[path pathExtension], NULL);
    CFStringRef MIMEType = UTTypeCopyPreferredTagWithClass (UTI, kUTTagClassMIMEType);
    CFRelease(UTI);
    if (!MIMEType) {
        return @"application/octet-stream";
    }
    return (__bridge NSString *)(MIMEType);
}
```

# 网络2

## NSOutputStream

- 文件流,文件输出流，可以输出到内存、硬盘、NSData

```objc
- (void)viewDidLoad {
    [super viewDidLoad];

    // 建立连接
    NSURL *url  = [NSURL URLWithString:@"http://123.123.123.123/resources/videos/minion_02.mp4"];
    [NSURLConnection connectionWithRequest:[NSURLRequest requestWithURL:url] delegate:self];
}
/**
 * 接收到响应的时候：创建一个空的文件
 */
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSHTTPURLResponse *)response
{
    // 获取服务器那里给出的建议名字   response.suggestedFilename);
    NSString *path = [[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:response.suggestedFilename];
    // 创建文件流
    self.stream = [[NSOutputStream alloc] initToFileAtPath:path append:YES];
    // 打开文件流
    [self.stream open];
}
/**
 * 接收到具体数据：马上把数据写入一开始创建好的文件
 */
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{
    // 参数1要求是bytes
    [self.stream write:[data bytes]  maxLength:data.length];
    NSLog(@"---");
}

- (void)connectionDidFinishLoading:(NSURLConnection *)connection
{
    // 关闭文件流
    [self.stream close];
}
```
## NSURLConnection和NSRunLoop
- 使用NSURLConnection创建的请求，其内部和NSRunLoop有关联，必须保证NSRunLoop处于运行状态，否则代理方法运行起来就会出问题。
- 如果要在子线程里创建请求，必须要手动启动NSRunLoop，并且要在方法使用结束手动关闭NSRunLoop。

```objc
// 建立连接
NSURL *url  = [NSURL URLWithString:@"http://123.123.123.123/resources/../images/minion_05.png"];
NSURLConnection *con = [NSURLConnection connectionWithRequest:[NSURLRequest requestWithURL:url] delegate:self];
// 决定代理方法在哪个队列中执行
[con setDelegateQueue:[[NSOperationQueue alloc] init]];
// 开启子线程runloop
self.runloop =  CFRunLoopGetCurrent();
CFRunLoopRun();
```
在代理方法中使用完毕，停止runloop

```objc
- (void)connectionDidFinishLoading:(NSURLConnection *)connection
{
    NSLog(@"--connectionDidFinishLoading--%@",[NSThread currentThread]);
    CFRunLoopStop(self.runloop);
}
```
## NSURLSession
- 这个是在iOS7之后推出的用于替代NSURLConnection的新类，**推荐掌握这个**。
- `NSURLSession` 主要由两部分组成，一个是Session实例对象，一个是任务。
- 使用步骤
	- 创建task（`dataTaskWithRequest`），启动task（`resume`）
	- 抽象类,使用其子类（`NSURLSessionDataTask`、`NSURLSessionDownloadTask`、`NSURLSessionUploadTask`）
	- 大小文件都一样，默认写入到tmp目录下面，下载完毕后要自己移动文件
- `NSURLSession` 代理
	- 初始化时设置代理` <NSURLSessionDataDelegate>`
	- 实现过程
		- 接收响应，指定响应方式：取消、下载、变为下载 `didReceivedResponse`
		- 接收数据`didReceivedData`
		- 接收完毕（成功和失败都会进入这个方法）`didComplete`
	- **这个可以实现大文件下载**
- 大文件断点下载
	- `NSURLSession` 的方法：`suspend、resume、cancel`
	- `resumeData` 保存暂停时程序数据状态,取消任务后要根据状态恢复下载
	- （不好 ，实现复杂）将下载的tmp文件保存到cachae，然后恢复现在时再从cache移动到临时文件
	- 下载失败后要从NSError中获取失败时的恢复数据
- 何为断点下载
	- 程序因意外事件终止，导致下载被停止，主要考虑用户突然关闭程序。
	- 可以将下载的数据同步到沙盒中，并且记录下载大小，以及记录已下载的文件，每次下载之前都进行扫描一番。
	- 如果下载一半就在请求头里指定要下载的范围
- 上传
	- `NSURLSessionUploadTast`
	- POST 请求设置注意`methodBody`为上传的参数`fromData`
	- `NSURLSessionConfiguration` 统一配置(比如可以在这里设置是否允许设置程序使用蜂窝移动网络)

### GET

```objc
NSURL *url  = [NSURL URLWithString:@"http://123.123.123.123/login?username=123&pwd=4324"];
// 1 获得NSURLSession单例对象
NSURLSession *session = [NSURLSession sharedSession];
// 2 创建任务
NSURLSessionDataTask *task = [session dataTaskWithRequest:[NSURLRequest requestWithURL:url] completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
    // 4 处理数据
    NSLog(@"----%@",[NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil]);
}];
// 3 启动任务
[task resume];
```
### POST

```objc
    // post请求
    NSURL *url  = [NSURL URLWithString:@"http://123.123.123.123/login?username=123&pwd=4324"];
    // 获得NSURLSession单例对象
    NSURLSession *session = [NSURLSession sharedSession];
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    // 设置请求头
    request.HTTPMethod = @"POST";
    // 设置请求体
    NSMutableData *body  = [NSMutableData data];
    [body appendData:[@"username=123&pwd=234" dataUsingEncoding:NSUTF8StringEncoding]];
    request.HTTPBody = body;
    // 创建任务
    NSURLSessionDataTask *task = [session dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        NSLog(@"----%@",[NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil]);
    }];
    // 启动任务
    [task resume];
```
### 下载

- 直接使用`downloadTaskWithURL:url`进行下载，不过下载成功的文件是放在tmp临时目录里面的，一定要及时把文件给移出来。

```objc
NSURL *url  = [NSURL URLWithString:@"http://123.123.123.123/resources/videos/minion_02.mp4"];
// 获得NSURLSession单例对象
NSURLSession *session = [NSURLSession sharedSession];
// 创建任务
NSURLSessionDownloadTask *task = [session downloadTaskWithURL:url completionHandler:^(NSURL *location, NSURLResponse *response, NSError *error) {
    NSString *filePath = [[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:response.suggestedFilename];
    // 这个方式下载的文件在tmp文件夹下，要把下载的文件移动到cache中永久保存,参数是 fileURLWithPath，看清了
    // loaction 表示在本地临时存储区的路径
    [[NSFileManager defaultManager] moveItemAtURL:location toURL:[NSURL fileURLWithPath:filePath] error:nil];
}];
// 启动任务
[task resume];
```
### 代理方式

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    NSURL *url  = [NSURL URLWithString:@"http://123.123.123.123/resources/videos/minion_02.mp4"];
    // 创建
    NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[[NSOperationQueue alloc] init]];

    // 创建任务
    NSURLSessionDataTask *task = [session dataTaskWithURL:url];

    // 启动任务
    [task resume];
}

// 接收服务器响应，必须手动执行后续的执行方式（允许接收数据还是不允许接收）
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler
{
    NSLog(@"%s",__func__);

    // 必须手动指定接下来的数据要不要接收
    completionHandler(NSURLSessionResponseAllow);
}
// 接收数据，多次调用
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data
{
    NSLog(@"%s",__func__);
}

// 下载完毕后调用，如果失败，看参数error的值
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    NSLog(@"%s",__func__);
}

typedef NS_ENUM(NSInteger, NSURLSessionResponseDisposition) {
    NSURLSessionResponseCancel = 0,/* Cancel the load, this is the same as -[task cancel] */
    NSURLSessionResponseAllow = 1,/* Allow the load to continue */
    NSURLSessionResponseBecomeDownload = 2,/* Turn this request into a download */
} NS_ENUM_AVAILABLE(NSURLSESSION_AVAILABLE, 7_0);

```
### 断点下载
- 通过自定义`NSURLSession` ，使用dataWithTask来进行下载，并且手动控制器下载文件的去向。
- 每次下载之前都要去沙盒读取已下载的文件，用于判断从哪里进行下载
- 主要方法如下：

```objc
- (NSURLSessionDataTask *)dataTask
{
    if (!_dataTask) {
        // 获取下载进度，直接从沙盒中读取文件长度
        NSInteger total = [[NSMutableDictionary dictionaryWithContentsOfFile:SLQDownloadFilePath][SLQFileName] integerValue];
        NSInteger current = [[[NSFileManager defaultManager] attributesOfItemAtPath:SLQFilePath error:nil][NSFileSize] integerValue];
        if (total && total == current ) {
            NSLog(@"已经下载完毕");
            return nil;
        }
        // 如果长度和
        NSURL *url  = [NSURL URLWithString:@"http://123.123.123.123/resources/videos/minion_02.mp4"];
        NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
        // 指定要从服务器获取的数据的范围
        NSString *range = [NSString stringWithFormat:@"bytes=%zd-",current];

        [request setValue:range forHTTPHeaderField:@"Range"];
        // 创建任务
        _dataTask = [self.session dataTaskWithRequest:request];
    }
    return  _dataTask;
}
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSHTTPURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler
{
//    NSLog(@"----%@",[response class]);
    // 开启输出流
    [self.stream open];
    // 计算总得数据长度，response会返回请求的数据长度，不包括已经下载的数据长度，所以要累加起来
    self.totalCount = [response.allHeaderFields[@"Content-Length"] integerValue] + SLQFileLength;
    NSString *name = response.suggestedFilename;
    // 存储总长度,将所要下载的文件长度保存起来
    NSMutableDictionary *dict = [NSMutableDictionary dictionaryWithContentsOfFile:SLQDownloadFilePath];
    if (dict == nil) {
        dict = [NSMutableDictionary dictionary];
    }
    dict[SLQFileName] = @(self.totalCount);
    [dict writeToFile:SLQDownloadFilePath atomically:YES];
    // 继续下载
    completionHandler(NSURLSessionResponseAllow);
}
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data
{
    // 写入数据到沙盒
    [self.stream write:[data bytes]maxLength:data.length];

    // 获取下载进度，直接从沙盒中读取文件长度
    NSLog(@"---%f",1.0 * SLQFileLength / self.totalCount);

}
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    // 清空
    self.dataTask = nil;
    [self.stream close];
    self.stream = nil;
}
```
### NSURLSession 上传文件
- 必须按照格式写，一个空格或者回车都不能多。

```objc
    // 一定要注意这个格式是固定的
    /* 文件参数格式
     --分割线\r\n
     Content-Disposition: form-data; name="file"; filename="文件名"\r\n
     Content-Type: 文件的MIMEType\r\n
     \r\n
     文件数据
     \r\n
     // 结束标记
     \r\n
     --分割线--\r\n
     \r\n
     */
     // 主要是参数第二个参数要传入 **`请求体`**
       [[self.session uploadTaskWithRequest:request fromData:body completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        NSLog(@"---%@",[NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil]);
    }] resume];
```

## AFNetworking

- GET\POST
    - `AFHTTPRequestManager`
    - `AFHTTPSessionManager`

```objc
- (void)GET
{
    // GET
    AFHTTPSessionManager *mgr = [AFHTTPSessionManager manager];
    // 将数据作为参数传入
    NSDictionary *dict = @{
                           @"username":@"12",
                           @"pwd":@"13"
                           };
    [mgr GET:[NSString stringWithFormat:@"http://123.123.123.123/login"] parameters:dict success:^(NSURLSessionDataTask *task, id responseObject) {
        NSLog(@"success:%@",responseObject);
    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        NSLog(@"failure:%@",error);
    }];
}
- (void)POST
{
    // POST
    AFHTTPSessionManager *mgr = [AFHTTPSessionManager manager];
    // 将数据作为参数传入
    NSDictionary *dict = @{
                           @"username":@"12",
                           @"pwd":@"13"
                           };
    [mgr POST:[NSString stringWithFormat:@"http://123.123.123.123/login"] parameters:dict success:^(NSURLSessionDataTask *task, id responseObject) {
        NSLog(@"success:%@",responseObject);
    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        NSLog(@"failure:%@",error);
    }];
}
```
- 文件上传：appendPartWithFileData:

```objc
- (void)upload
{
    // 文件上传
    AFHTTPSessionManager *mgr = [AFHTTPSessionManager manager];
    // 将数据作为参数传入
    [mgr POST:@"http://123.123.123.123/upload" parameters:nil constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
        // formdata 为要上传的数据
//        [formData appendPartWithFileURL:[NSURL fileURLWithPath:@"/Users/song/Desktop/test.png"] name:@"file" error:nil];
        [formData appendPartWithFileData:[NSData dataWithContentsOfFile:@"/Users/song/Desktop/test.png"] name:@"file" fileName:@"wer.png" mimeType:@"image/png"];
    } success:^(NSURLSessionDataTask *task, id responseObject) {
        NSLog(@"success:%@",responseObject);
    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        NSLog(@"failure:%@",error);
    }];
}
```
- 文件下载
    - 下载文件需要返回一个保存路径，还需要手动启动resume

```objc
- (void)download
{
    // 下载
    AFHTTPSessionManager *mgr = [AFHTTPSessionManager manager];

    [[mgr downloadTaskWithRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://123.123.123.123/resources/../images/minion_02.png"]] progress:nil destination:^NSURL *(NSURL *targetPath, NSURLResponse *response) {
        // 下载文件需要返回一个保存路径，还需要手动启动resume
        NSString *path = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0];
        return [NSURL fileURLWithPath:[path stringByAppendingPathComponent:response.suggestedFilename]];
    } completionHandler:^(NSURLResponse *response, NSURL *filePath, NSError *error) {
        NSLog(@"%@",filePath.path);
    }] resume];
}
```
- 默认是解析json，如果想解析xml，需要指定管理器的解析器为xml
    - 如果解析其他类型的文件，就将`responseSerializer`属性设置为`ADHTTPResonseSericlizer`，服务器返回什么就接受什么类型的数据。

```objc
-(void)returnType
{
    // 默认返回的数据时JSON，如果想返回XML，设置属性responseSerializer
    // 如果想返回服务器上文件本来的类型，设置AFHTTPResponseSerializer
    AFHTTPSessionManager *mgr = [AFHTTPSessionManager manager];
    // responseSerializer 用来解析服务器返回的数据
    mgr.responseSerializer = [AFHTTPResponseSerializer serializer]; // 直接使用“服务器本来返回的数据”，不做任何解析
    // 告诉AFN，以XML形式解析服务器返回的数据
    //    mgr.responseSerializer = [AFXMLParserResponseSerializer serializer];
    // 将数据作为参数传入
    [mgr GET:[NSString stringWithFormat:@"http://123.123.123.123/resources/../images/minion_02.png"] parameters:nil success:^(NSURLSessionDataTask *task,id response) {
        NSLog(@"success:%zd",[response length]);
    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        NSLog(@"failure:%@",error);
    }];
}
```

- 手机联网状态
	- 手机联网状态：`AFNetWorkReachabityManager`
	- 苹果自带：`Reachability` ，通过通知监听系统状态

- 手机联网状态：`AFNetWorkReachabityManager`

```objc
- (void)monitor
{
    // 监控网络状态
    AFNetworkReachabilityManager *mgr  = [AFNetworkReachabilityManager sharedManager];
    // 网络状态改变就会调用这个block
    [mgr setReachabilityStatusChangeBlock:^(AFNetworkReachabilityStatus status) {
        NSLog(@"网络状态改变：%zd",status);
    }];
    // 打开监听器
    [mgr startMonitoring];

    /*
    typedef NS_ENUM(NSInteger, AFNetworkReachabilityStatus) {
     AFNetworkReachabilityStatusUnknown          = -1, // 未知
     AFNetworkReachabilityStatusNotReachable     = 0, // 未联网
     AFNetworkReachabilityStatusReachableViaWWAN = 1, // 蜂窝网络
     AFNetworkReachabilityStatusReachableViaWiFi = 2, // wifi
     };
     */
}
```

- 手机联网状态：`Reachability`
    - 手机的状态改变，会给系统发送通知，所以可以添加监听器，接收这个通知。

```objc
/**通知*/
@property (nonatomic, strong) Reachability *reach;

- (void)viewDidLoad
{
    [super viewDidLoad];
    // 添加通知
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(getNetworkStatus) name:kReachabilityChangedNotification object:nil];
    // 接收通知
    self.reach = [Reachability reachabilityForInternetConnection];
    [self.reach startNotifier];
}
- (void)getNetworkStatus
{
    /*
     typedef enum : NSInteger {
     NotReachable = 0, // 网络不可知
     ReachableViaWiFi, // WIFI
     ReachableViaWWAN  // 移动网络
     } NetworkStatus;
     */
    // 获取手机网络状态
    if([Reachability reachabilityForLocalWiFi].currentReachabilityStatus != NotReachable)
    {
        NSLog(@"wifi");
    }
    else if([Reachability reachabilityForInternetConnection].currentReachabilityStatus != NotReachable)
    {
        NSLog(@"3G?4G");
    }
    else
    {
        NSLog(@"Nothing at all!");
    }
}
- (void)dealloc
{
    // 停止监听器
    [self.reach startNotifier];
    // 移除通知
      [[NSNotificationCenter defaultCenter] removeObserver:self name:kReachabilityChangedNotification object:nil];
}
```
## MD5加密
- 使用:主要是对用户的敏感信息进行加密
	- 对输入信息生成唯一的128位散列值（32个字符）
	- 根据输出值，不能得到原始的明文，即其过程不可逆
- MD5改进
	- 加盐（Salt）：在明文的固定位置插入随机串，然后再进行MD5
	- 先加密，后乱序：先对明文进行MD5，然后对加密得到的MD5串的字符进行乱序
	- 等等，就是让信息更加复杂

## HTTPS
- 使用https要实现代理方法 `didReceiveChallenge`

- 1 创建HTTPS链接

```objc
        _dataTask = [self.session dataTaskWithURL:[NSURL URLWithString:@"https://www.apple.com/"] completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
            NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
        }];
```
- 2 实现代理方法 `didReceiveChallenge`

```objc
/** 代理方法
 * challenge ： 挑战、质询
 * completionHandler : 通过调用这个block，来告诉URLSession要不要接收这个证书
 */
- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential *))completionHandler
{
    // NSURLSessionAuthChallengeDisposition ： 如何处理这个安全证书
    // NSURLCredential ：安全证书
//    NSLog(@"%@",challenge);
    if (![challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]) {
        return;
    }
    if(completionHandler)
    {
        // 利用这个block说明使用这个证书
        completionHandler(NSURLSessionAuthChallengeUseCredential,challenge.proposedCredential);
    }
}
```
## UIWebView
- 显示网页数据
- 代理方法`<UIWebViewDelegate>`
	- `shouldStartLoadWithRequest`: 请求之前判断是否允许访问(过滤某些网址)
	- 属性UIScrollView可以控制滚动范围
	- `loadHTMLString`
	- `loadData：`
	- 可以加载网络资源和本地资源
	- `scalesPageToFit` 屏幕自适应
	- `dataDetectorTypes` 自动检测网页中出现的电话号码，网址等，添加下划线和链接。

```objc
// 始发送请求（加载数据）时调用这个方法
- (void)webViewDidStartLoad:(UIWebView *)webView;
// 请求完毕（加载数据完毕）时调用这个方法
- (void)webViewDidFinishLoad:(UIWebView *)webView;
// 请求错误时调用这个方法
- (void)webView:(UIWebView *)webView didFailLoadWithError:(NSError *)error;
// UIWebView在发送请求之前，都会调用这个方法，如果返回NO，代表停止加载请求，返回YES，代表允许加载请求
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType;
```

```objc
/*
* 每当webView即将发送一个请求之前，都会调用这个方法
* 返回YES：允许加载这个请求
* 返回NO：禁止加载这个请求
*/
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{
     NSLog(@"%s",__func__);
    if ([request.URL.absoluteString containsString:@"life"]) {
        return NO;
    }
    return YES;
}
```
## JS介绍

### HTML5
- html（内容） + CSS（样式） + JS（动态效果、事件交互）
- 常用JS函数
	-  `-alert(10);  // 弹框`
	- `document.getElementById(‘test’); // 根据ID获得某个DOM元素`
- JS和OC通信
- oc执行js
	- `stringByEvaluatingJavaScriptFromString`
	- JS 函数
	- `function`
- JS执行OC
	- 通过代理法方法 `shouldStartLoadWithRequest`
	- 在js函数中调用 `loaction.href = 'slq://sendMessage_?参数1&参数2';`
	- 传递参数的话，在方法后边写入一符号（_、@等）标识要传递参数，然后参数之间也要加入符号分割

### OC执行JS
- 是用OC执行那个JS脚本
- `stringByEvaluatingJavaScriptFromString`

```objc
   [webView stringByEvaluatingJavaScriptFromString:@"alert(100)"];
   // 调用JS中的函数，
    NSLog(@"%@",[webView stringByEvaluatingJavaScriptFromString:@"login();"]);
    self.title = [webView stringByEvaluatingJavaScriptFromString:@"document.title;"];
```

### JS执行OC
- 通过代理法方法 shouldStartLoadWithRequest

- 无参数传递

```objc
/**
 * 通过这个方法完成JS调用OC
 * JS和OC交互的第三方框架：WebViewJavaScriptBridge
 */
 // location.href = 'slq://call';
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
{
    //
    NSLog(@"%@",request.URL);
    NSString *url = request.URL.absoluteString;
    NSString *pre = @"slq://";
    if([url hasPrefix:pre])
    {
        // 调用OC方法
//        NSLog(@"调用OC方法");
        NSString *method = [url substringFromIndex:pre.length];
        // NSSelectorFromString 将字符串转换成方法名
        [self performSelector:NSSelectorFromString(method) withObject:nil];
        return NO;
    }
//    NSLog(@"发送请求");
    return YES;
}
```
- 2个参数
	- 使用'_'代替':','？'区分参数和函数名，'&'区分参数

```objc
// location.href = 'slq://call2_number2_?&100';
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
{
    NSString *url = request.URL.absoluteString;
    NSString *pre = @"slq://";
    if([url hasPrefix:pre])
    {
        // 调用OC方法
        NSString *method = [url substringFromIndex:pre.length];
        // method = call2_number2_?200&300
        // 分割字符串
        NSArray *arr = [method componentsSeparatedByString:@"?"];
        // call2:number2:
        NSString *methodName = [[arr firstObject] stringByReplacingOccurrencesOfString:@"_" withString:@":"];
        // 200&300
        NSString *paramStr = [arr lastObject];

        NSArray *params = nil;
        if (arr.count == 2 && [paramStr containsString:@"&"]) {
            params = [paramStr componentsSeparatedByString:@"&"];
        }

        NSString *param1 = [params firstObject];
        NSString *param2 = params.count <= 1 ? nil : [params lastObject];
        NSLog(@"%@",methodName);

        [self performSelector:NSSelectorFromString(methodName) withObject:param1 withObject:param2]; // 两个参数

//        [self performSelector:NSSelectorFromString(methodName) withObject:para];// 一个参数
        return NO;
    }
    return YES;
}
```

- 3个参数
	- 如果有3个以上参数，只能使用方法签名的方式来确定传递参数

```objc
- (id)performSelector:(SEL)selector withObjects:(NSArray *)params
{
    // 设置方法签名
    NSMethodSignature *sig = [[self class] instanceMethodSignatureForSelector:selector];
    //
    if (sig == nil) {
        NSLog(@"方法没找到");
    }
    // NSInvocation 包装对象利用一个NSInvocation对象包装一次方法调用（方法调用者、方法名、方法参数、方法返回值）
    NSInvocation *invo = [NSInvocation invocationWithMethodSignature:sig];
    invo.target = self;
    invo.selector = selector;

    // 设置参数
    NSInteger paramNum = sig.numberOfArguments - 2; // 包含两个隐含参数：self and _cmd
    // 取最小值作为参数,
    paramNum = MIN(paramNum, params.count);
    for (int i = 0 ; i < paramNum; i ++) {
        id obj = params[i];
        if ([obj isKindOfClass:[NSNull class]]) {
            continue;
        }
        [invo setArgument:&obj atIndex:i + 2]; // 从第三个参数开始
    }
    // 设置回调
    [invo invoke];

    // 返回值
    id returnVal = 0;
    if (sig.methodReturnLength) {
        [invo getReturnValue:&returnVal];
    }
    return returnVal;
}
```

### 程序崩溃处理

- 在appdelegate中判断

```objc
void handleException(NSException *exception)
{
    [[UIApplication sharedApplication].delegate performSelector:@selector(handle)];
}

- (void)handle
{
    UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"哈哈" message:@"崩溃了把" delegate:self cancelButtonTitle:@"好的" otherButtonTitles:nil, nil];
    [alertView show];

    // 重新启动RunLoop
    [[NSRunLoop currentRunLoop] addPort:[NSPort port] forMode:NSDefaultRunLoopMode];
    [[NSRunLoop currentRunLoop] run];
}

- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
{
    NSLog(@"-------点击了好的");
    exit(0);
}

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    // 设置捕捉异常的回调
    NSSetUncaughtExceptionHandler(handleException);

    return YES;
}
```

### 去除Xcode编译警告

- 如果对于某些警告需要屏蔽，需要找到这个警告 的代号

![](../images/查找警告对应的代号.gif)

```objc
 // 去除Xcode编译警告
//#pragma clang diagnostic push // 开始
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
//#pragma clang diagnostic pop // 结束
```

### 异常处理

- 如果方法名错误，**抛出异常** `@throw`
- **捕获异常** `@try @catch @finally`
- 程序崩溃分析报告
	- 如果要监视整个程序的异常，可以在main函数中@try
	- 提前已处理异常，`NSSetUncaughtExceptionHandler()`
	- 崩溃统计
		- 友盟
		- Flurry
		- Crashlytics


