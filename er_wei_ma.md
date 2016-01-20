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
    - 有一点需要注意，就是敏感区域设置的坐标系是反转的，也就是说xy互换，wh互换。

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
- 控制器 


```objc

#import <UIKit/UIKit.h>
typedef void(^returnQrcodeBlock)(NSString *code);
@interface SLQQrcodeViewController : UIViewController
@property (nonatomic ,copy)returnQrcodeBlock qrcodeBlock;
@end


```

```objc
//
//  ViewController.m
//  二维码
//
//  Created by Christian on 15/8/21.
//  Copyright (c) 2015年 slq. All rights reserved.
//

#import "SLQQrcodeViewController.h"
#import <AVFoundation/AVFoundation.h>

static const char *kScanQRCodeQueueName = "ScanQRCodeQueue";
@interface SLQQrcodeViewController ()<AVCaptureMetadataOutputObjectsDelegate>
@property (strong, nonatomic) UIView *sanFrameView;
@property (nonatomic) AVCaptureSession *captureSession;
@property (nonatomic) AVCaptureVideoPreviewLayer *videoPreviewLayer;
@property (nonatomic) BOOL lastResut;
@end

@implementation SLQQrcodeViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    _lastResut = YES;
    UIView *scan = [[UIView alloc] initWithFrame:self.view.bounds];
    _sanFrameView = scan;
    UIButton *back = [[UIButton alloc] initWithFrame:CGRectMake(20, 20, 100, 100)];
    [back setImage:[UIImage imageNamed:@"back_camera_btn"] forState:UIControlStateNormal];
    [back addTarget:self action:@selector(back) forControlEvents:UIControlEventTouchUpInside];
    UIButton *light = [[UIButton alloc] initWithFrame:CGRectMake(SCREEN_WIDTH - 120, 20, 100, 100)];
    [light setImage:[UIImage imageNamed:@"flash_camera_btn"] forState:UIControlStateNormal];
    [light addTarget:self action:@selector(lightOn:) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:_sanFrameView];
    [self.view addSubview:back];
    [self.view addSubview:light];
    
    // 黑色背景区域
    UIView *line = [[UIView alloc] initWithFrame:CGRectMake(0, 0, SCREEN_WIDTH, (SCREEN_HEIGHT - 200)/2)];
    line.backgroundColor = [UIColor blackColor];
    line.alpha = 0.5;
    [self.view addSubview:line];
    UIView *line2 = [[UIView alloc] initWithFrame:CGRectMake(0, (SCREEN_HEIGHT - 200)/2+200, SCREEN_WIDTH, (SCREEN_HEIGHT - 200)/2)];
    line2.backgroundColor = [UIColor blackColor];
    line2.alpha = 0.5;
    [self.view addSubview:line2];
    
    UIView *line3 = [[UIView alloc] initWithFrame:CGRectMake(0,  (SCREEN_HEIGHT - 200)/2, (SCREEN_WIDTH - 200)/2, 200)];
    line3.backgroundColor = [UIColor blackColor];
    line3.alpha = 0.5;
    [self.view addSubview:line3];
    UIView *line4 =  [[UIView alloc] initWithFrame:CGRectMake((SCREEN_WIDTH - 200)/2+200,  (SCREEN_HEIGHT - 200)/2, (SCREEN_WIDTH - 200)/2, 200)];
    line4.backgroundColor = [UIColor blackColor];
    line4.alpha = 0.5;
    [self.view addSubview:line4];
    
    UILabel *title = [[UILabel alloc] initWithFrame:CGRectMake(0, (SCREEN_HEIGHT - 200)/2+200+10, SCREEN_WIDTH, 44)];
    title.text = @"请对准二维码进行扫描";
    title.textAlignment = NSTextAlignmentCenter;
    [self.view addSubview:title];
    
    
    
    UIView *orangeLine = [[UIView alloc] initWithFrame:CGRectMake((SCREEN_WIDTH - 200)/2, (SCREEN_HEIGHT - 200)/2, 200, 1)];
    orangeLine.backgroundColor = [UIColor orangeColor];
    [self.view addSubview:orangeLine];
    
    [UIView animateWithDuration:1.2 delay:0 options:UIViewAnimationOptionAutoreverse | UIViewAnimationOptionRepeat animations:^{
        orangeLine.transform = CGAffineTransformMakeTranslation(0, 200);
    } completion:^(BOOL finished) {
        
    }];
    [self startReading];
}
- (void)dealloc
{
    [self stopReading];
}

- (void)back
{
    [self.navigationController popViewControllerAnimated:YES];
}

-(void)lightOn:(UIButton *)btn
{
    if (btn.isSelected == NO) {
        btn.selected = YES;
        [self systemLightSwitch:YES];
    }
    else
    {
        btn.selected = NO;
        [self systemLightSwitch:NO];
    }
}

- (BOOL)startReading
{
    //    [_button setTitle:@"停止" forState:UIControlStateNormal];
    // 获取 AVCaptureDevice 实例
    NSError * error;
    AVCaptureDevice *captureDevice = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    // 初始化输入流
    AVCaptureDeviceInput *input = [AVCaptureDeviceInput deviceInputWithDevice:captureDevice error:&error];
    if (!input) {
        NSLog(@"%@", [error localizedDescription]);
        return NO;
    }
    // 创建会话
    _captureSession = [[AVCaptureSession alloc] init];
    // 添加输入流
    [_captureSession addInput:input];
    // 初始化输出流
    AVCaptureMetadataOutput *captureMetadataOutput = [[AVCaptureMetadataOutput alloc] init];
    // 添加输出流
    [_captureSession addOutput:captureMetadataOutput];
    // 扫描区域
    captureMetadataOutput.rectOfInterest = CGRectMake((SCREEN_HEIGHT-200)/2.0 /SCREEN_HEIGHT, (SCREEN_WIDTH-200)/2.0 /SCREEN_WIDTH,200.0/SCREEN_HEIGHT,200.0/SCREEN_WIDTH);
    // 创建dispatch queue.
    dispatch_queue_t dispatchQueue;
    dispatchQueue = dispatch_queue_create(kScanQRCodeQueueName, NULL);
    [captureMetadataOutput setMetadataObjectsDelegate:self queue:dispatchQueue];
    // 设置元数据类型 AVMetadataObjectTypeQRCode
    [captureMetadataOutput setMetadataObjectTypes:[NSArray arrayWithObject:AVMetadataObjectTypeQRCode]];
    
    // 创建输出对象
    _videoPreviewLayer = [[AVCaptureVideoPreviewLayer alloc] initWithSession:_captureSession];
    [_videoPreviewLayer setVideoGravity:AVLayerVideoGravityResizeAspect];
    [_videoPreviewLayer setFrame:_sanFrameView.layer.bounds];
    [_sanFrameView.layer addSublayer:_videoPreviewLayer];

    // 开始会话
    [_captureSession startRunning];
    
    return YES;
}

- (void)stopReading
{
    //    [_button setTitle:@"开始" forState:UIControlStateNormal];
    // 停止会话
    [_captureSession stopRunning];
    _captureSession = nil;
}

- (void)reportScanResult:(NSString *)result
{
    [self stopReading];
    if (!_lastResut) {
        return;
    }
    _lastResut = NO;
    if (_qrcodeBlock) {
        _qrcodeBlock(result);
    }
    [self.navigationController popViewControllerAnimated:YES];
}

- (void)systemLightSwitch:(BOOL)open
{
    AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    if ([device hasTorch]) {
        [device lockForConfiguration:nil];
        if (open) {
            [device setTorchMode:AVCaptureTorchModeOn];
        } else {
            [device setTorchMode:AVCaptureTorchModeOff];
        }
        [device unlockForConfiguration];
    }
}

#pragma AVCaptureMetadataOutputObjectsDelegate

-(void)captureOutput:(AVCaptureOutput *)captureOutput didOutputMetadataObjects:(NSArray *)metadataObjects
      fromConnection:(AVCaptureConnection *)connection
{
    if (metadataObjects != nil && [metadataObjects count] > 0) {
        AVMetadataMachineReadableCodeObject *metadataObj = [metadataObjects objectAtIndex:0];
        NSString *result;
        if ([[metadataObj type] isEqualToString:AVMetadataObjectTypeQRCode]) {
            result = metadataObj.stringValue;
        } else {
            NSLog(@"不是二维码");
        }
        [self performSelectorOnMainThread:@selector(reportScanResult:) withObject:result waitUntilDone:NO];
    }
}


- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}


@end
```




# 内存分析
- 静态内存
	- 直接分析代码，不运行程序
	- menu->Product->Analyze
- 动态内存


