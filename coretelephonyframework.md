# CoreTelephony.framework

- 可以使用这个框架拿到手机的一些基本信息
- 比如说sim卡制式

```objc
#import "ViewController.h"
@import CoreTelephony;

@interface ViewController ()

{
    CTCallCenter *center_; // In order to avoid the formation of retain cycle and the statement of a variable, pointing to the object receiving call center
}
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    // To obtain and output of mobile phone operator information
    [self getCarrierInfo];
    
    // Monitoring the call information
    CTCallCenter *center = [[CTCallCenter alloc] init];
    center_ = center;
    center.callEventHandler = ^(CTCall *call) {
        NSSet *curCalls = center_.currentCalls;
        NSLog(@"current calls:%@", curCalls);
        NSLog(@"call:%@", [call description]);
    };
    
    /*
     {
     Carrier name: [中国联通]
     Mobile Country Code: [460]
     Mobile Network Code:[01]
     ISO Country Code:[cn]
     Allows VOIP? [YES]
     }
     */
    
}
- (void)getCarrierInfo {
    // Gets the operator information
    CTTelephonyNetworkInfo *info = [[CTTelephonyNetworkInfo alloc] init];
    CTCarrier *carrier = info.subscriberCellularProvider;
    NSLog(@"carrier:%@", [carrier description]);
    
    // If operators change will update operator output
    info.subscriberCellularProviderDidUpdateNotifier = ^(CTCarrier *carrier) {
        NSLog(@"carrier:%@", [carrier description]);
    };
    
    // Output mobile phone data service information
    NSLog(@"Radio Access Technology:%@", info.currentRadioAccessTechnology);
    // outpu Technology:CTRadioAccessTechnologyHSDPA
}

/*
 * Radio Access Technology values
 */
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyGPRS          __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyEdge          __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyWCDMA         __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyHSDPA         __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyHSUPA         __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyCDMA1x        __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyCDMAEVDORev0  __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyCDMAEVDORevA  __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyCDMAEVDORevB  __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyeHRPD         __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyLTE           __OSX_AVAILABLE_STARTING(__MAC_NA,__IPHONE_7_0);
```

