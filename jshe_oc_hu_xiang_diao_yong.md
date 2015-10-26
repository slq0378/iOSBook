# JS和OC相互调用

## OC调用JS

- 通过OC调用HTML，主要通过webView执行`stringByEvaluatingJavaScriptFromString:`
- 也可以 返回自己想要的值，这样就把JS中的数据出传递到OC中了

```
#import "ViewController.h"

@interface ViewController ()<UIWebViewDelegate>
@property (weak, nonatomic) IBOutlet UIWebView *webView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    //加载网页
    NSURL *url = [[NSBundle mainBundle] URLForResource:@"index" withExtension:@"html"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    [self.webView loadRequest:request];
}

#pragma mark - 操作网页
-(void)webViewDidFinishLoad:(UIWebView *)webView{
    // 删除
    NSString *str1 = @"var word = document.getElementById('word');";
    NSString *str2 = @"word.remove();";
    // 执行JS脚本
    [webView  stringByEvaluatingJavaScriptFromString:str1];
    [webView  stringByEvaluatingJavaScriptFromString:str2];

    // 更改
    NSString *str3 = @"var change = document.getElementsByClassName('change')[0];"
                       "change.innerHTML = '好你的哦!';";
    [webView stringByEvaluatingJavaScriptFromString:str3];

    // 插入图片
    NSString *str4 =@"var img = document.createElement('img');"
                     "img.src = 'img_01.jpg';"
                     "img.width = '160';"
                     "img.height = '80';"
                     "document.body.appendChild(img);";
    [webView stringByEvaluatingJavaScriptFromString:str4];

}
```

## JS调用OC
- html中处理

```
<html>
   <head>
      <meta charset="UTF-8">
   </head>
   <body>
      <button onclick="getImage();">访问相册</button>

      <script>
          function getImage(){
             /*这里对这个函数定义一个协议头，在OC里进行解析，当点击这个按钮时，就会调用这个方法，然后这个事件会通过代理方法传递到OC中，在OC中再次响应点击的处理事件，打开相册*/
             window.location.href = 'slq://getImage';
          }
      </script>
   </body>
</html>
```

- OC中处理

```
- (void)viewDidLoad {
    [super viewDidLoad];
    //加载网页
    NSURL *url = [[NSBundle mainBundle] URLForResource:@"index" withExtension:@"html"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    [self.webView loadRequest:request];

}
/**
 *  加载webView的任何请求都会经过这个方法，返回yes表示允许加载，NO表示禁止加载
 */
-(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{
    NSString *str = request.URL.absoluteString;
        NSLog(@"%@", str);
    NSRange range = [str rangeOfString:@"slq://"];
    if (range.location != NSNotFound) {
   	     // 截取字符串，获取方法名
        NSString *method = [str substringFromIndex:range.location + range.length];
        // 字符串转换成方法名
        SEL sel = NSSelectorFromString(method);
        [self performSelector:sel];
    }
    return YES;
}

// 访问相册
- (void)getImage{
    UIImagePickerController *pickerImg = [[UIImagePickerController alloc]init];
    pickerImg.sourceType = UIImagePickerControllerSourceTypePhotoLibrary;

    [self presentViewController:pickerImg animated:YES completion:nil];
}
```
