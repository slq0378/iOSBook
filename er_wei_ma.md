# 二维码
## 生成二维码
- 二维码可以存放纯文本、名片或者URL
- 生成二维码的步骤：
    - 导入CoreImage框架
    - 通过滤镜CIFilter生成二维码
        - 1、创建过滤器
        - 2、恢复滤镜的默认属性
        - 3、设置内容
        - 4、获取输出文件
        - 5、显示二维码
- 代码实现 `CoreImage`

```objc
    // 二维码的生成

    // 1、创建过滤器
    CIFilter *filter = [CIFilter filterWithName:@"CIQRCodeGenerator"];

    // 2、恢复滤镜的默认属性
    [filter setDefaults];

    // 3、设置内容
    NSString *str = @"这是一个二维码的生成结果";
    NSData *data = [str dataUsingEncoding:NSUTF8StringEncoding];

    // 使用KVO设置属性
    [filter setValue:data forKey:@"inputMessage"];

    // 4、获取输出文件
    CIImage *outputImage = [filter outputImage];

    // 5、显示二维码
    self.imageView.image = [UIImage imageWithCIImage:outputImage];
```
- 这样显示的图片不是很清晰，可以自己重绘图片
    - 重新生成高清图片:网上找即可

```objc
/**
 *  根据CIImage生成指定大小的UIImage
 *
 *  @param image CIImage
 *  @param size  图片宽度
 */
- (UIImage *)createNonInterpolatedUIImageFormCIImage:(CIImage *)image withSize:(CGFloat) size
{
    CGRect extent = CGRectIntegral(image.extent);
    CGFloat scale = MIN(size/CGRectGetWidth(extent), size/CGRectGetHeight(extent));

    // 1.创建bitmap;
    size_t width = CGRectGetWidth(extent) * scale;
    size_t height = CGRectGetHeight(extent) * scale;
    CGColorSpaceRef cs = CGColorSpaceCreateDeviceGray();
    CGContextRef bitmapRef = CGBitmapContextCreate(nil, width, height, 8, 0, cs, (CGBitmapInfo)kCGImageAlphaNone);
    CIContext *context = [CIContext contextWithOptions:nil];
    CGImageRef bitmapImage = [context createCGImage:image fromRect:extent];
    CGContextSetInterpolationQuality(bitmapRef, kCGInterpolationNone);
    CGContextScaleCTM(bitmapRef, scale, scale);
    CGContextDrawImage(bitmapRef, extent, bitmapImage);

    // 2.保存bitmap到图片
    CGImageRef scaledImage = CGBitmapContextCreateImage(bitmapRef);
    CGContextRelease(bitmapRef);
    CGImageRelease(bitmapImage);
    return [UIImage imageWithCGImage:scaledImage];
}
```
- 还有就是设置内容为网址时，如果带有协议头的话，会自动打开网页。
    - `NSString *str = @"http://www.baidu.com";`

## 扫描二维码
- `AVFoundation`框架
-  二维码的扫描过程
    - 1、创建捕捉会话`AVCaptureSession`
    - 2、添加输入设备(数据从摄像头输入) `AVCaptureDevice` `AVCaptureDeviceInput`
    - 3、添加输出数据(示例对象-->类对象-->元类对象-->根元类对象) `AVCaptureMetadataOutput`
        - 3.1.设置输入元数据的类型(类型是二维码数据) `setMetadataObjectTypes`
    - 4、添加扫描图层 `AVCaptureVideoPreviewLayer`
    - 5、开始扫描 `startRunning`
    - 6、实现回调代理方法，获取扫描结果 `captureOutput: :`

```objc
#import "ViewController.h"
#import <AVFoundation/AVFoundation.h>

@interface ViewController () <AVCaptureMetadataOutputObjectsDelegate>
/**显示图层*/
@property (nonatomic, strong) AVCaptureVideoPreviewLayer *layer;
/**捕捉会话*/
@property (nonatomic, strong) AVCaptureSession *session;

@end

@implementation ViewController


- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    // 二维码的扫描
    // 1、创建捕捉会话
    AVCaptureSession *session = [[AVCaptureSession alloc] init];
    self.session = session;

    // 2.添加输入设备(数据从摄像头输入)
    AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    AVCaptureDeviceInput *input = [AVCaptureDeviceInput deviceInputWithDevice:device error:nil];
    [session addInput:input];

    // 3、添加输出数据(示例对象-->类对象-->元类对象-->根元类对象)
    AVCaptureMetadataOutput *output = [[AVCaptureMetadataOutput alloc] init];
    [output setMetadataObjectsDelegate:self queue:dispatch_get_main_queue()];
    [session addOutput:output];

    // 3.1.设置输入元数据的类型(类型是二维码数据)
    [output setMetadataObjectTypes:@[AVMetadataObjectTypeQRCode]];

    // 4、添加扫描图层
    AVCaptureVideoPreviewLayer *layer = [AVCaptureVideoPreviewLayer layerWithSession:session];
    layer.frame = self.view.bounds;
    [self.view.layer addSublayer:layer];
    self.layer = layer;

    // 5、开始扫描
    [session startRunning];
}

/**
 *	实现output的回调方法
 */
- (void)captureOutput:(AVCaptureOutput *)captureOutput didOutputMetadataObjects:(NSArray *)metadataObjects fromConnection:(AVCaptureConnection *)connection
{
    /* 数组metadataObjects中存放数据如下
     (
    "<AVMetadataMachineReadableCodeObject: 0x17002dac0, type=\"org.iso.QRCode\", bounds={ 0.4,0.3 0.2x0.3 }>corners { 0.4,0.7 0.5,0.7 0.5,0.4 0.4,0.3 }, time 6788633292291, stringValue \"http://weixin.qq.com/r/SDiEnCHEyXy2rWX-921a\""
    )*/

    if (metadataObjects.count > 0) {
        // 获取最终的读取结果
        AVMetadataMachineReadableCodeObject *object = [metadataObjects lastObject];
        NSLog(@"%@",object.stringValue);
        [self.session stopRunning];
        [self.layer removeFromSuperlayer];
    }
    else
    {
        NSLog(@"没有扫描到数据");
    }
}
@end
```

- 全部源代码

```objc

```

```objc
```


# 内存分析
- 静态内存
	- 直接分析代码，不运行程序
	- menu->Product->Analyze
- 动态内存


