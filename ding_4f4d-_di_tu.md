# 定位、地图
- 定位 `CoreLocation`
- 地图 `MapKit`

## CoreLocation
- `CLLocationManager`
- 在获取用户信息时会弹出获取用户隐私的对话框

![](/images/Snip20150823_7.png)

- iOS设置里每个应用的位置授权信息


- 可以在info.plist的文件中添加一个描述信息
-
- 6.0
	- 添加info.list 的key:`Privacy - Location Usage Description`
- 8.0 适配
	- 前后台（永久授权）：`requestAlwaysAuthorization`
		- 必须需要手动添加info.list的key `NSLocationAlwaysUsageDescription`
	- 前台（临时授权+顶部蓝个长条）：`requestWhenInUserAuthorization`
		- 必须需要手动添加info.list的key `NSLocationWhenInUseUsageDescription`
		- 必须实现后台获取：在工程-Capablities->Background Modes -> Location updates

		![](/images/Snip20150823_8.png)

```objc
#import "ViewController.h"
#import <CoreLocation/CoreLocation.h>

@interface ViewController () <CLLocationManagerDelegate>

/**定位*/
@property (nonatomic, strong) CLLocationManager *mgr;

@end

@implementation ViewController

#pragma mark - 懒加载
- (CLLocationManager *)mgr
{
    if (!_mgr) {
        // 1、创建定位管理器
        _mgr = [[CLLocationManager alloc] init];
        // 2、设置代理
        _mgr.delegate = self;

         // 每隔多米定位一次
        _mgr.distanceFilter = 100;
        /**
         kCLLocationAccuracyBestForNavigation // 最适合导航
         kCLLocationAccuracyBest; // 最好的
         kCLLocationAccuracyNearestTenMeters; // 10m
         kCLLocationAccuracyHundredMeters; // 100m
         kCLLocationAccuracyKilometer; // 1000m
         kCLLocationAccuracyThreeKilometers; // 3000m
         */
        // 精确度越高, 越耗电, 定位时间越长
        _mgr.desiredAccuracy = kCLLocationAccuracyBest;


        // iOS 8.0 适配
        if([[UIDevice currentDevice].systemVersion floatValue] >= 8.0)
        {
            NSLog(@"系统版本iOS 8.0 +");
            // 前后台定位授权
            [_mgr requestAlwaysAuthorization];
            // 前台定位授权(默认情况下,不可以在后台获取位置, 勾选后台模式 location update, 但是 会出现蓝条)
//            [_mgr requestWhenInUseAuthorization];
        }

    }
    return  _mgr;
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    NSLog(@"%s",__func__);
    // 3、开始更新位置信息
    if ([CLLocationManager locationServicesEnabled])
    {
        NSLog(@"开始更新位置信息");
        [self.mgr startUpdatingLocation];
    }
    else
    {
        NSLog(@"位置不可用");
    }
}

/**
 *	更新到位置之后调用
 */
- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations
{
    NSLog(@"定位位置：%@",[locations lastObject]);
}

/**
 *	获取定位信息的状态
 */
- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status
{
    switch (status) {
            // 用户还未决定
        case kCLAuthorizationStatusNotDetermined:
        {
            NSLog(@"用户还未决定");
            break;
        }
            // 问受限
        case kCLAuthorizationStatusRestricted:
        {
            NSLog(@"访问受限");
            break;
        }
            // 定位关闭时和对此APP授权为never时调用
        case kCLAuthorizationStatusDenied:
        {
            // 定位是否可用（是否支持定位或者定位是否开启）
            if([CLLocationManager locationServicesEnabled])
            {
                NSLog(@"定位开启，但被拒");
            }else
            {
                NSLog(@"定位关闭，不可用");
            }
            //            NSLog(@"被拒");
            break;
        }
            // 获取前后台定位授权
        case kCLAuthorizationStatusAuthorizedAlways:
            //        case kCLAuthorizationStatusAuthorized: // 失效，不建议使用
        {
            NSLog(@"获取前后台定位授权");
            break;
        }
            // 获得前台定位授权
        case kCLAuthorizationStatusAuthorizedWhenInUse:
        {
            NSLog(@"获得前台定位授权");
            break;
        }
        default:
            break;
    }
}

@end
```

- 9.0 适配
	- 前台（临时授权+顶部蓝个长条）：`requestWhenInUserAuthorization`
		- 添加后台模式，设置属性 `allocsBackgroundLocationUpdate`
		- 添加key
		- 设置后台模式
	- 前后台（永久授权）：`requestAlwaysAuthorization`
		- 添加后台模式，设置属性`allocsBackgroundLocationUpdate`
		- 添加key
		- 设置后台模式

```objc
      // 允许后台获取用户位置(iOS9.0)
    if([[UIDevice currentDevice].systemVersion floatValue] >= 9.0)
    {
         // 一定要勾选后台模式 location updates
        _mgr.allowsBackgroundLocationUpdates = YES;
    }
```
	- 方法：`requestLocation`
		- Request a single location update.

- CLLocation 解析
	- coordinate:经纬度 （latitude：维度）、（longitude：径度）
	- altitude：海拔高度
	- course: 角度（正北）
	- speed : 速度

```objc
#pragma mark - CLLocationManagerDelegate
- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray<CLLocation *> *)locations
{
//    NSLog(@"获取位置：%@",[locations lastObject]);
    CLLocation *loc = [locations lastObject];
    /** CLLocation
     <+37.60549252,-122.42794789> +/- 5.00m (speed 33.07 mps / course 351.56) @ 15/8/23 中国标准时间 下午9:37:08
     CLLocation 解析
     coordinate:经纬度 （latitude：维度）、（longitude：径度）
     altitude：海拔高度
     course: 角度（正北）
     speed : 速度
     */
//    NSLog(@"经纬度:%.3f,%.3f",loc.coordinate.latitude,loc.coordinate.longitude);
//    NSLog(@"海拔高度:%.2f",loc.altitude);
//      NSLog(@"角度:%.2f",loc.course);
//      NSLog(@"速度:%.2f",loc.speed);

    /**
     *  场景演示:打印当前用户的行走方向,偏离角度以及对应的行走距离,
     例如:”北偏东 30度  方向,移动了8米”
     */
    NSString *direction = nil;
    // 获取方向 ，除以90的方向
    switch ((NSInteger)loc.course / 90) {
        case 0:
            direction = @"北偏东";
            break;
        case 1:
            direction = @"东偏南";
            break;
        case 2:
            direction = @"南偏西";
            break;
        case 3:
            direction = @"西偏北";
            break;
        default:
            direction = @"跑偏了";
            break;
    }

    // 获取角度，对90取余的角度
    NSInteger angle = (NSInteger)loc.course % 90;
    if(angle == 0)
    {
        direction =[NSString stringWithFormat:@"正%@",[direction substringToIndex:1]];
    }

    // 获取移动距离，首先要记录上一次的位置，然后进行比较
    CLLocationDistance distance = 0;
    if (self.oldLocation) {
        distance = [loc distanceFromLocation:self.oldLocation];
    }
    self.oldLocation = loc;
    NSString *result = nil;
    result = (angle == 0) ? [NSString stringWithFormat:@"%@方向，移动了%.2f米",direction,distance]:[NSString stringWithFormat:@"%@ %zd方向，移动了%.2f米",direction,angle,distance];;
    NSLog(@"%@",result);
}
```

## 指南针

- `startUpdatingHeading`

```objc
#pragma mark - 懒加载
- (CLLocationManager *)mgr
{
    if (!_mgr) {
        // 1、创建定位管理器
        _mgr = [[CLLocationManager alloc] init];
        // 2、设置代理
        _mgr.delegate = self;
        // 每隔多少度更新一次
        _mgr.headingFilter = 2;
    }
    return  _mgr;
}

- (void)viewDidLoad
{
    [super viewDidLoad];

     // 开始监听设备朝向
    [self.mgr startUpdatingHeading];
}


- (void)locationManager:(CLLocationManager *)manager didUpdateHeading:(CLHeading *)newHeading
{

    /**
     *  CLHeading
     *  magneticHeading : 磁北角度
     *  trueHeading : 真北角度
     */

    // 这里对图片进行旋转
    // 转换成弧度
    double angle = newHeading.magneticHeading / 180.0 * M_PI;
    [UIView animateWithDuration:0.2 animations:^{
        self.compassView.transform = CGAffineTransformMakeRotation(-angle);
        NSLog(@"角度%f",angle);

    }];
}
```

## 区域监听
- `CLLocationManager`
- `startMonitoringRegion:`

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    // 设置监听区域
    CLLocationCoordinate2D center = {21.13, 123.456};
    CLCircularRegion *region = [[CLCircularRegion alloc] initWithCenter:center radius:1000 identifier:@"wansisdfsa"];

    [self.mgr startMonitoringForRegion:region];

}

- (void)locationManager:(CLLocationManager *)manager didEnterRegion:(CLRegion *)region
{
    NSLog(@"进入监听区域");
}
- (void)locationManager:(CLLocationManager *)manager didExitRegion:(CLRegion *)region
{
      NSLog(@"离开监听区域");
}
```

## 地理编码（反地理编码）
- 位置和经纬度转换
- `CLGeoCoder`

```objc
#pragma mark - 懒加载
- (CLGeocoder *)geo
{
    if (!_geo) {
        // 1、创建定位管理器
        _geo = [[CLGeocoder alloc] init];
    }
    return  _geo;
}
- (IBAction)geoCoder:(id)sender {
    // 地理编码：由地址获取到经纬度等
    [self.geo geocodeAddressString:@"盛达商务园" completionHandler:^(NSArray<CLPlacemark *> * _Nullable placemarks, NSError * _Nullable error) {

        /**
         *  CLPlacemark
         location : 位置对象
         addressDictionary : 地址字典
         name : 地址全称
         */

        if (error == nil) {
            [placemarks enumerateObjectsUsingBlock:^(CLPlacemark * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
                NSLog(@"%@\n%5@", obj.name,obj.location);
            }];
        }
        else
        {
             NSLog(@"error--%@", error.localizedDescription);
        }
    }];
}
- (IBAction)reverseGeoCoder:(id)sender {
    // 反地理编码：由经纬度获取地址
    CLLocation *loc = [[CLLocation alloc] initWithLatitude:23.13293100 longitude:113.37592400];
    [self.geo reverseGeocodeLocation:loc completionHandler:^(NSArray<CLPlacemark *> * _Nullable placemarks, NSError * _Nullable error) {
        if (error == nil) {
            [placemarks enumerateObjectsUsingBlock:^(CLPlacemark * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
                NSLog(@"%@\n%5@", obj.name,obj.location);
            }];
        }
        else
        {
            NSLog(@"error--%@", error.localizedDescription);
        }
    }];
}
```

- iOS9.0 布局 UIStackView
- 参考文章
<http://www.cocoachina.com/ios/20150623/12233.html>

## 第三方
- 介绍一种 `LocationManager`
- 具体使用参考官方文章

```objc
    INTULocationManager *locMgr = [INTULocationManager sharedInstance];
    [locMgr requestLocationWithDesiredAccuracy:INTULocationAccuracyCity
                                       timeout:10.0
                          delayUntilAuthorized:YES  // This parameter is optional, defaults to NO if omitted
                                         block:^(CLLocation *currentLocation, INTULocationAccuracy achievedAccuracy, INTULocationStatus status) {
                                             if (status == INTULocationStatusSuccess) {
                                                 NSLog(@"获取成功：%@",currentLocation);
                                             }
                                             else {
                                                 NSLog(@"error");
                                             }
                                         }];
```


## 属性和成员变量


- 属性可以对数据进行过滤(setter 方法)
- 懒加载（getter方法）
- 优缺点
	1. 防止对象被提前创建
	2. 防止对象重复创建
	3. 防止对象使用时,还没被创建
	4. 可以在懒加载方法里面,进行初始化操作

# `MapKit`

- 基本属性 `MapView`

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    /*
     MKMapTypeStandard = 0, // 基本
     MKMapTypeSatellite,    // 卫星
     MKMapTypeHybrid        // 混合
     MKMapTypeSatelliteFlyover NS_ENUM_AVAILABLE(10_11, 9_0) // 3D卫星,
     MKMapTypeHybridFlyover NS_ENUM_AVAILABLE(10_11, 9_0),  // 3D混合
     */
    self.mapView.mapType = MKMapTypeSatellite;
//
//    self.mapView.showsScale = YES;
//    self.mapView.showsBuildings = YES;//
    [self lM];
    self.mapView.showsUserLocation = YES;
    self.mapView.userTrackingMode = MKUserTrackingModeFollowWithHeading;
}
```


- 代理方法 `MKMapViewDelegate`

```objc
#pragma mark - MKMapViewDelegate
- (void)mapView:(MKMapView *)mapView didUpdateUserLocation:(MKUserLocation *)userLocation
{
    NSLog(@"%@",userLocation);
    userLocation.title = @"小码哥";
    userLocation.subtitle = @"大神一班";
    // 显示在最中心
//    [self.mapView setCenterCoordinate:userLocation.location.coordinate animated:YES];


    // 设置地图显示区域：跨度
    MKCoordinateSpan span = MKCoordinateSpanMake(0.051109, 0.034153);
    MKCoordinateRegion region = MKCoordinateRegionMake(userLocation.location.coordinate, span);
    [self.mapView setRegion:region animated:YES];
}
```

##  大头针
	- `addAnnotation:`
	- `removeAnnotation:`
- 自定义大头针模型
-  要遵守协议 <MKAnnotation>

```objc
#import <Foundation/Foundation.h>
#import <MapKit/MapKit.h>

@interface SLQAnnotation : NSObject <MKAnnotation>
// 位置
@property (nonatomic, assign) CLLocationCoordinate2D coordinate;

// 标题和子标题
@property (nonatomic, copy, nullable) NSString *title;
@property (nonatomic, copy, nullable) NSString *subtitle;

/** 类型 */
@property (nonatomic, assign) NSInteger type;
@end
```
- 自定义大头针视图

- `MKPinAnnnotationView` 是 `MKAnnotationView`的子类

```objc
- (MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id<MKAnnotation>)annotation
{
    static NSString *inden = @"annotation";
    MKPinAnnotationView *pin = (MKPinAnnotationView *)[mapView dequeueReusableAnnotationViewWithIdentifier:inden];

    if (pin == nil) {
        pin = [[MKPinAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:inden];
    }

    pin.annotation = annotation;

    // 设置是否弹出标注
    pin.canShowCallout = YES;

    // 设置大头针颜色
    pin.pinTintColor = [UIColor blackColor];

    // 从天而降
    pin.animatesDrop = YES;

    // 设置大头针图片(系统大头针无效)
    //    pin.image = [UIImage imageNamed:@"category_5"];
    // 可拖动大头针
    pin.draggable = YES;

    return pin;
}
```

- `MKAnnnotationView`
![](/images/Snip20150824_1.png)

```objc
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    // 1、获取触摸点
    CGPoint curPoint = [[touches anyObject] locationInView:self.mapView];

    // 2、转换触摸点坐标位经纬度
    CLLocationCoordinate2D coordinate = [self.mapView convertPoint:curPoint toCoordinateFromView:self.mapView];

    // 3、添加大头针
    [self addAnnotation:coordinate];
}
#pragma mark - MKMapViewDelegate
- (MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id<MKAnnotation>)annotation
{
    static NSString *AnnoID = @"annotation";
    MKAnnotationView *view = [mapView dequeueReusableAnnotationViewWithIdentifier:AnnoID];
    if (view == nil) {
        view = [[MKAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:AnnoID];
    }

    view.annotation = annotation;
    // 设置属性

    // 设置是否弹出标注
    view.canShowCallout = YES;
//    SLQAnnotation *an = (SLQAnnotation *)annotation;
//    NSString *imageName = [NSString stringWithFormat:@"01%zd",an.type];
//    view.image = [UIImage imageNamed:imageName];
    // 大头针可以拖动
    view.draggable = YES;

    // 左右两边显示的view
    view.leftCalloutAccessoryView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"010"]];
    view.rightCalloutAccessoryView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"011"]];

    // 设置详细标签
    view.detailCalloutAccessoryView = [[UISwitch alloc] init];

    //
//    view.animatesDrop = YES;

    return view;
}
```

## 利用系统地图进行导航

-



### 3D视图导航

- `MKMapCamera`

```objc
// 3D导航视图
MKMapCamera *camera = [MKMapCamera cameraLookingAtCenterCoordinate:CLLocationCoordinate2DMake(23.132931, 113.375924) fromEyeCoordinate:CLLocationCoordinate2DMake(23.135931, 113.375924) eyeAltitude:10];

self.mapView.camera = camera;
```

### 地图快照
- `MKMapSnapshotter`

```objc
    // 地图快照设置选项
    MKMapSnapshotOptions *options = [[MKMapSnapshotOptions alloc] init];
    // 设置快照区域
    options.region = self.mapView.region;
    options.showsBuildings = YES;
    // 设置快照尺寸
    options.size = CGSizeMake(1000, 2000);
    options.scale = [UIScreen mainScreen].scale;
    // 创建快照
    MKMapSnapshotter *shoter = [[MKMapSnapshotter alloc] initWithOptions:options];
    // 截图完成后回调
    [shoter startWithCompletionHandler:^(MKMapSnapshot * _Nullable snapshot, NSError * _Nullable error) {
        // 结果在这里
        if(!error)
        {
            UIImage *image = snapshot.image;
            NSData *data = UIImageJPEGRepresentation(image, 1);
            [data writeToFile:@"/Users/song/Desktop/map.png" atomically:YES];
        }
        else
        {
            NSLog(@"截图出错，请重来");
        }
    }];
```

### 获取导航路线

```objc
- (void)beginNav
{
    [self.geoCoder geocodeAddressString:@"广州" completionHandler:^(NSArray<CLPlacemark *> * _Nullable placemarks, NSError * _Nullable error) {
        // 广州地标
        CLPlacemark *gz = [placemarks firstObject];

        [self.geoCoder geocodeAddressString:@"上海" completionHandler:^(NSArray<CLPlacemark *> * _Nullable placemarks, NSError * _Nullable error) {
            // 上海地标
            CLPlacemark *sh = [placemarks firstObject];

            [self beginNavWithStartLocation:gz endLocation:sh];
            NSLog(@"开始导航");
        }];
    }];

}
// 使用两个地标来规划路线
- (void)beginNavWithStartLocation:(CLPlacemark *)start endLocation:(CLPlacemark *)end
{
    // 起始位置
    MKPlacemark *startP = [[MKPlacemark alloc] initWithPlacemark:start];
    MKMapItem *startI = [[MKMapItem alloc] initWithPlacemark:startP];
    // 目的位置
    MKPlacemark *endP = [[MKPlacemark alloc] initWithPlacemark:end];
    MKMapItem *endI = [[MKMapItem alloc] initWithPlacemark:endP];

    NSDictionary *dict = @{
                           // 导航方式
                           MKLaunchOptionsDirectionsModeKey : MKLaunchOptionsDirectionsModeDriving,

                           // 地图类型
                           MKLaunchOptionsMapTypeKey : @(MKMapTypeHybrid),

                           // 是否显示交通
                           MKLaunchOptionsShowsTrafficKey : @(YES)
                           };

    [MKMapItem openMapsWithItems:@[startI,endI] launchOptions:dict];
}
```

- 开始地点、结束地点

- `MKDirection`

```objc
// 获取路线具体信息
- (void)getRouteWithBeginPL:(CLPlacemark *)start andEndPL:(CLPlacemark *)end
{
    MKDirectionsRequest *request = [[MKDirectionsRequest alloc] init];
    // 起始位置
    MKPlacemark *startP = [[MKPlacemark alloc] initWithPlacemark:start];
    MKMapItem *startI = [[MKMapItem alloc] initWithPlacemark:startP];
    request.source = startI;

    // 目的位置
    MKPlacemark *endP = [[MKPlacemark alloc] initWithPlacemark:end];
    MKMapItem *endI = [[MKMapItem alloc] initWithPlacemark:endP];
    request.destination = endI;

    // 路线方向
    MKDirections *direction = [[MKDirections alloc] initWithRequest:request];

    [direction calculateDirectionsWithCompletionHandler:^(MKDirectionsResponse * _Nullable response, NSError * _Nullable error) {
        /**
         *  MKDirectionsResponse
         routes : 路线数组MKRoute

         */
        /**
         *  MKRoute
         name : 路线名称
         distance : 距离
         expectedTravelTime : 预期时间
         polyline : 折线(数据模型)
         steps
         */
        /**
         *  steps <MKRouteStep *>
         instructions : 行走提示
         */

        if (error == nil) {
            [response.routes enumerateObjectsUsingBlock:^(MKRoute * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
                NSLog(@"%@---%zd---%f", obj.name, obj.expectedTravelTime, obj.distance);

                [obj.steps enumerateObjectsUsingBlock:^(MKRouteStep * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
                      NSLog(@"%@", obj.instructions);
                }];
            }];
        }
        else
        {
            NSLog(@"路线错误");
        }

    }];
}
```

### 添加覆盖层

- 绘制路线,利用地图的代理方法,返回对应的图层渲染
- `MKPolyLine` `MKPolyLineRenderer`

```objc
// 获取路线具体信息
- (void)getRouteWithBeginPL:(CLPlacemark *)start andEndPL:(CLPlacemark *)end
{
    // 添加圆形模型
    MKCircle *circleStart = [MKCircle circleWithCenterCoordinate:start.location.coordinate radius:1000];
    [self.mapView addOverlay:circleStart];

    MKCircle *circleEnd = [MKCircle circleWithCenterCoordinate:end.location.coordinate radius:1000];
    [self.mapView addOverlay:circleEnd];

    MKDirectionsRequest *request = [[MKDirectionsRequest alloc] init];
    // 起始位置
    MKPlacemark *startP = [[MKPlacemark alloc] initWithPlacemark:start];
    MKMapItem *startI = [[MKMapItem alloc] initWithPlacemark:startP];
    request.source = startI;

    // 目的位置
    MKPlacemark *endP = [[MKPlacemark alloc] initWithPlacemark:end];
    MKMapItem *endI = [[MKMapItem alloc] initWithPlacemark:endP];
    request.destination = endI;

    // 路线方向
    MKDirections *direction = [[MKDirections alloc] initWithRequest:request];

    [direction calculateDirectionsWithCompletionHandler:^(MKDirectionsResponse * _Nullable response, NSError * _Nullable error) {
        /**
         *  MKDirectionsResponse
         routes : 路线数组MKRoute

         */
        /**
         *  MKRoute
         name : 路线名称
         distance : 距离
         expectedTravelTime : 预期时间
         polyline : 折线(数据模型)
         steps
         */
        /**
         *  steps <MKRouteStep *>
         instructions : 行走提示
         */

        if (error == nil) {
            [response.routes enumerateObjectsUsingBlock:^(MKRoute * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {

                // 添加一个覆盖层数据模型
                [self.mapView addOverlay:obj.polyline];
            }];
        }
        else
        {
            NSLog(@"路线错误");
        }

    }];
}


- (MKOverlayRenderer *)mapView:(MKMapView *)mapView rendererForOverlay:(id<MKOverlay>)overlay
{
    // 在两头绘制一个圆
    if([overlay isKindOfClass:[MKCircle class]])
    {
        MKCircleRenderer *circleR = [[MKCircleRenderer alloc] initWithOverlay:overlay];

        circleR.fillColor = [UIColor cyanColor];
        circleR.alpha = 0.5;

        return circleR;
    }

    // 绘制路线
    else if([overlay isKindOfClass:[MKPolylineRenderer class]])
    {
        MKPolylineRenderer *render = [[MKPolylineRenderer alloc] initWithOverlay:overlay];
        render.lineWidth = 10;
        render.strokeColor = [UIColor redColor];
        return render;
    }
    return nil;
}
```

## 集成百度地图

- 参考官方文档
- <http://developer.baidu.com/map/index.php?title=ios-navsdk/guide/helloworld>
