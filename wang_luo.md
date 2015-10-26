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



