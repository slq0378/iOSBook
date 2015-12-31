# 南沙公安局


## AFNetworking简单封装

```objc
#import <AFNetworking.h>
#import "AFHTTPSessionManager.h"

typedef void (^successBlock)(id data);
typedef void (^failedBlock)(NSError *error);

@interface AZHTTPClient : AFHTTPSessionManager

+ (instancetype)shareInstance;

// GET请求方法
+ (void)getUrlString:(NSString *)url
           withParam:(NSDictionary *)param
    withSuccessBlock:(successBlock)success
     withFailedBlock:(failedBlock)failed;


// POST请求方法
+ (void)postUrlString:(NSString *)url
            withParam:(NSDictionary *)param
     withSuccessBlock:(successBlock)success
      withFailedBlock:(failedBlock)failed;
@end
```

```objc
#import "AZHTTPClient.h"

@implementation AZHTTPClient

+ (instancetype)shareInstance {
    
    static AZHTTPClient *client = nil;
    
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        client = [[AZHTTPClient alloc] init];
        client.securityPolicy.allowInvalidCertificates = YES;
        client.securityPolicy.validatesDomainName = NO;
        client.responseSerializer.acceptableContentTypes = [NSSet setWithArray:@[@"text/html",@"application/x-msdownload"]];
    });
    
    return client;
    
}

+ (void)getUrlString:(NSString *)url
           withParam:(NSDictionary *)param
    withSuccessBlock:(successBlock)success
     withFailedBlock:(failedBlock)failed {
 
    [[self shareInstance] GET:url parameters:param progress:^(NSProgress * _Nonnull downloadProgress) {
        
    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        success(responseObject);
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        failed(error);
    }];

}

+ (void)postUrlString:(NSString *)url
            withParam:(NSDictionary *)param
     withSuccessBlock:(successBlock)success
      withFailedBlock:(failedBlock)failed {
    
    [[self shareInstance] POST:url parameters:param constructingBodyWithBlock:^(id<AFMultipartFormData>  _Nonnull formData) {
        
    } progress:^(NSProgress * _Nonnull uploadProgress) {
        
    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        success(responseObject);
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        failed(error);
    }];


    
}
@end
```

## AFNetworking 使用出错(2.6以后版本)
- `NSURLSession/NSURLConnection HTTP load failed (kCFStreamErrorDomainSSL, -9813)`

```objc
        client = [[AZHTTPClient alloc] init];
        client.securityPolicy.allowInvalidCertificates = YES;
        client.securityPolicy.validatesDomainName = NO;
        client.responseSerializer.acceptableContentTypes = [NSSet setWithArray:@[@"text/html",@"application/x-
```
