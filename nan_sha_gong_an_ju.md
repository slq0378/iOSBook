# 南沙公安局

http://www.umeng.com/

naihaijingwu@163.com
naihaijingwu123
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



- 上传与下载

```objc
- (void)downloadUserImage
{
    NSString *userId = [CommenDefault sharedCommenDefault].userInfo.userId;
    NSString *userPsw = [[MD5 MD5Encrypt:[CommenDefault sharedCommenDefault].userInfo.password] uppercaseString];
    NSString *uid = userId;
    NSString *headtype = @"1";
    
    NSMutableDictionary *para = [[NSMutableDictionary alloc] init];
    para[@"userId"] = userId;
    para[@"userPsw"] = userPsw;
    para[@"uid"] = uid;
    para[@"headtype"] = headtype;
    
    NSString *url = [NSString stringWithFormat:@"https://%@:82/frame/webInteface.do?method=downHeadPic",NHBaseURL];
    NSLog(@"url:%@",url);
    
    [[AZHTTPClient shareInstance] GET:url parameters:para success:^(AFHTTPRequestOperation *operation, id responseObject) {
        if (responseObject == nil) {
            if (operation.responseData) {
                
                self.userImage = [UIImage imageWithData:operation.responseData];
                [self.mainTable reloadData];
            }
        }
       else if ([responseObject[@"result"] integerValue] == 0) {
            NSLog(@"下载头像失败-%@",responseObject[@"errorMsg"]);
            [MBProgressHUD hideHUDForView:self.view animated:YES];
            [MBProgressHUD showError:responseObject[@"errorMsg"] toView:self.view];
        }
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        NSLog(@"上传头像失败-%@",error);
        [MBProgressHUD hideHUDForView:self.view animated:YES];
        [MBProgressHUD showError:@"上传失败" toView:self.view];
    }];
}

- (void)uploadUserImage
{
    NSString *userId = [CommenDefault sharedCommenDefault].userInfo.userId;
    NSString *userPsw = [[MD5 MD5Encrypt:[CommenDefault sharedCommenDefault].userInfo.password] uppercaseString];
    NSString *uid = userId;
    
    NSMutableDictionary *para = [[NSMutableDictionary alloc] init];
    para[@"userId"] = userId;
    para[@"userPsw"] = userPsw;
    para[@"uid"] = uid;
    UIImage *img = [UIImage imageNamed:@"camera"];
    // 截图
    NSData *shotImageData = UIImageJPEGRepresentation(img, 0.1);
    [AZFunction saveImage:shotImageData withidStr:@"dddd"];
    
    
    NSString *url = [NSString stringWithFormat:@"https://%@:82/frame/webInteface.do?method=upHeadPic",NHBaseURL];
    NSLog(@"url:%@",url);
    [[AZHTTPClient shareInstance] POST:url parameters:para constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
        // 地图截图
        NSData *data = [AZFunction getImageWithIdStr:@"dddd"];
        [formData appendPartWithFileData:data name:@"headpic" fileName:@"headpic" mimeType:@"image/jpg"];
        
    } success:^(AFHTTPRequestOperation *operation, id responseObject) {
        if ([responseObject[@"result"] integerValue] == 0) {
            NSLog(@"上传头像失败");
            [MBProgressHUD hideHUDForView:self.view animated:YES];
            [MBProgressHUD showError:responseObject[@"errorMsg"] toView:self.view];
        }
        else
        {
            NSLog(@"上传登记成功，responseObject = %@",responseObject);
            [MBProgressHUD hideHUDForView:self.view animated:YES];
            [MBProgressHUD showSuccess:@"上传成功" toView:self.view];
        }
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        NSLog(@"上传头像失败-%@",error);
        [MBProgressHUD hideHUDForView:self.view animated:YES];
        [MBProgressHUD showError:@"上传失败" toView:self.view];
    }];
}

```