# 南沙公安局
## AFNetworking 使用出错
- `NSURLSession/NSURLConnection HTTP load failed (kCFStreamErrorDomainSSL, -9813)`

```objc
       client = [[AZHTTPClient alloc] init];
        client.securityPolicy.allowInvalidCertificates = YES;
        client.securityPolicy.validatesDomainName = NO;
        client.responseSerializer.acceptableContentTypes = [NSSet setWithArray:@[@"text/html",@"application/x-
```