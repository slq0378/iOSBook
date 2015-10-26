## 多线程
- NSThread
- GCD
    - 队列
        - 并发队列
            - 全局队列
            - 自己创建
        - 串行队列
            - 自己创建
            - 主队列
    - 任务：block
    - 函数
        - sync：同步函数
        - async：异步函数
    - 单例模式
- NSOperation
- RunLoop
    - 同一时间只能选择一个模式运行
    - 常用模式
        - Default：默认
        - Tracking：拖拽UIScrollView

## 网络
### HTTP请求
- GET请求

```objc
// URL
NSString *urlStr = @"http://xxxxx.com";
urlStr = [urlStr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];

NSURL *url = [NSURL URLWithString:urlStr];

// 请求
NSURLRequest *request = [NSURLRequest requestWithURL:url];
```

- POST请求

```objc
// URL
NSString *urlStr = @"http://xxxxx.com";
urlStr = [urlStr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];

NSURL *url = [NSURL URLWithString:urlStr];

// 请求
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
request.HTTPMethod = @"POST";
request.HTTPBody = [@"username=123&pwd=123" dataUsingEncoding:NSUTF8StringEncoding];
```

### 数据解析
- JSON
    - NSJSONSerialization
- XML
    - SAX: NSXMLParser
    - DOM: GDataXML

### NSURLConnection（iOS9已经过期）
- 同步方法
- 异步方法-block
- 异步方法-代理

### NSURLSession
- 发送一般的GET\POST请求

```objc
NSURLSessionDataTask *task = [[NSURLSession sharedSession] dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {

}];

[task resume];
```

- 下载文件 - 不需要离线断点

```objc
NSURLSessionDownloadTask *task = [[NSURLSession sharedSession] downloadTaskWithRequest:request completionHandler:^(NSURL *location, NSURLResponse *response, NSError *error) {

}];

[task resume];
```

- 下载文件 - 需要离线断点
- 开启任务

```objc
// 创建session
NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[[NSOperationQueue alloc] init]];

// 设置请求头（告诉服务器第1024个字节开始下载）
[request setValue:@"bytes=1024-" forHTTPHeaderField:@"Range"];

// 创建任务
NSURLSessionDataTask *task = [session dataTaskWithRequest:request];

// 开始任务
[task resume];
```

- 实现代理方法

```objc
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler
{
    // 打开NSOutputStream

    // 获得文件的总长度

    // 存储文件的总长度

    // 回调
    completionHandler(NSURLSessionResponseAllow);
}

- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data
{
    // 利用NSOutputStream写入数据

    // 计算下载进度
}

- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    // 关闭并且清空NSOutputStream

    // 清空任务task
}
```

- 文件上传
    - 设置请求头
    - 拼接请求体
        - 文件参数
        - 其他参数（非文件参数）
    - NSURLSessionUploadTask

```objc
NSURLSessionUploadTask *task = [[NSURLSession sharedSession] uploadTaskWithRequest:request fromData:body completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {

}];

[task resume];
```

### AFN
- GET

```objc
// AFHTTPSessionManager内部包装了NSURLSession
AFHTTPSessionManager *mgr = [AFHTTPSessionManager manager];

NSDictionary *params = @{
                         @"username" : @"520it",
                         @"pwd" : @"520it"
                         };

[mgr GET:@"http://120.25.226.186:32812/login" parameters:params success:^(NSURLSessionDataTask *task, id responseObject) {
    NSLog(@"请求成功---%@", responseObject);
} failure:^(NSURLSessionDataTask *task, NSError *error) {
    NSLog(@"请求失败---%@", error);
}];
```

- POST

```objc
// AFHTTPSessionManager内部包装了NSURLSession
AFHTTPSessionManager *mgr = [AFHTTPSessionManager manager];

NSDictionary *params = @{
                         @"username" : @"520it",
                         @"pwd" : @"520it"
                         };

[mgr POST:@"http://120.25.226.186:32812/login" parameters:params success:^(NSURLSessionDataTask *task, id responseObject) {
    NSLog(@"请求成功---%@", responseObject);
} failure:^(NSURLSessionDataTask *task, NSError *error) {
    NSLog(@"请求失败---%@", error);
}];
```

- 文件上传

```objc
AFHTTPSessionManager *mgr = [AFHTTPSessionManager manager];

[mgr POST:@"http://120.25.226.186:32812/upload" parameters:@{@"username" : @"123"}
    constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
    // 在这个block中设置需要上传的文件
//            NSData *data = [NSData dataWithContentsOfFile:@"/Users/xiaomage/Desktop/placeholder.png"];
//            [formData appendPartWithFileData:data name:@"file" fileName:@"test.png" mimeType:@"image/png"];

//            [formData appendPartWithFileURL:[NSURL fileURLWithPath:@"/Users/xiaomage/Desktop/placeholder.png"] name:@"file" fileName:@"xxx.png" mimeType:@"image/png" error:nil];

        [formData appendPartWithFileURL:[NSURL fileURLWithPath:@"/Users/xiaomage/Desktop/placeholder.png"] name:@"file" error:nil];
} success:^(NSURLSessionDataTask *task, id responseObject) {
    NSLog(@"-------%@", responseObject);
} failure:^(NSURLSessionDataTask *task, NSError *error) {

}];
```

### UIWebView
- 加载请求
- JS和OC的互相调用
- 利用NSInvocation实现performSelector无限参数
- 异常捕捉
    - NSException
    - 崩溃统计（第三方）
        - 友盟
        - Flurry
        - Crashlytics



