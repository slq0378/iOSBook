# 南沙公安局


## AFNetworking
## AFNetworking 使用出错(2.6以后版本)
- `NSURLSession/NSURLConnection HTTP load failed (kCFStreamErrorDomainSSL, -9813)`

```objc
        client = [[AZHTTPClient alloc] init];
        client.securityPolicy.allowInvalidCertificates = YES;
        client.securityPolicy.validatesDomainName = NO;
        client.responseSerializer.acceptableContentTypes = [NSSet setWithArray:@[@"text/html",@"application/x-
```
