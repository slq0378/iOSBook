# 格式化时间显示


```objc
/** 显示方式
 今年
    今天
        1分钟内
            刚刚
        1小时内
            xx分钟前
        其他
            xx小时前
    昨天
        昨天 18:56:34
    其他
        06-23 19:56:23

 非今年
    2014-05-08 18:45:30
 */
```

- 实现方法

```objc
- (NSString *)create_time
{
    NSDateFormatter *fmt = [[NSDateFormatter alloc] init];
    fmt.dateFormat = @"yyyy-MM-dd HH-mm-ss";
    NSDate *createDate = [fmt dateFromString:_create_time];
    // 获取时间差
    NSDateComponents  *comps = [[NSDate date] deltaFrom:createDate];

    if (createDate.isThisYear) { // 如果是今年
        if(createDate.isToday) { // 如果是今天


            if(comps.hour > 1) { // 其他情况，多少小时之前
                return [NSString stringWithFormat:@"%zd小时前",comps.hour];
            }
            else if(comps.minute > 1) { // 小于1小时,xx分钟前
                return [NSString stringWithFormat:@"%zd分钟前",comps.minute];
            }
            else { // 小于1分钟,刚刚
                return @"刚刚";
            }
        }
        else if(createDate.isYesterday){ // 如果是昨天，昨天 12：23：34
            fmt.dateFormat = @"昨天 HH-mm-ss";
            return [fmt stringFromDate:createDate];
        }
        else { // 其他，显示 05-09 12：22：43
            fmt.dateFormat = @"MM-dd HH-mm-ss";
            return [fmt stringFromDate:createDate];
        }
    }
    else { // 不是今年,直接显示全部 2013-2-3 04：09：34
        return _create_time;
    }
}
```
- 实现日期分类方法

```objc
#import "NSDate+SLQDate.h"

@implementation NSDate (SLQDate)
/**
 *	返回日期差值
 */
- (NSDateComponents *)deltaFrom:(NSDate *)from
{
    NSCalendar *cal = [NSCalendar currentCalendar];

    NSCalendarUnit unit = NSCalendarUnitYear | NSCalendarUnitMonth | NSCalendarUnitDay | NSCalendarUnitHour | NSCalendarUnitMinute | NSCalendarUnitSecond;
   return [cal components:unit fromDate:from  toDate:self options:0];
}

/**
 *	是否是今年
 */
- (BOOL)isThisYear
{
    // 首先直接判断year，但是注意这个特殊情况
    // 2014-12-31 --- 2015-1-1
    // 获取当前时间
    NSCalendar *calendar = [NSCalendar currentCalendar];

    // 获得当前年
    NSInteger nowYear = [calendar component:NSCalendarUnitYear fromDate:[NSDate date]];
    // 获得传入参数年
    NSInteger fromYear = [calendar component:NSCalendarUnitYear fromDate:self];
    return nowYear == fromYear;
}
/**
 *	是否是今天,如果是同一天，那么天数之前的字符串应该相等
 */
- (BOOL)isToday
{
    // 比对年月日
    NSDateFormatter *fmt = [[NSDateFormatter alloc] init];
    fmt.dateFormat = @"yyyy-MM-dd";

    // 今天
    NSString *nowDay = [fmt stringFromDate:[NSDate date]];
    // 传入日期
    NSString *fromDay = [fmt stringFromDate:self];

    return [nowDay isEqualToString:fromDay];
}
/**
 *	是否是昨天
 */
- (BOOL)isYesterday
{
    // 比对年月日
    NSDateFormatter *fmt = [[NSDateFormatter alloc] init];
    fmt.dateFormat = @"yyyy-MM-dd";

    // 获取时间
    NSDate *now = [fmt dateFromString:([fmt stringFromDate:[NSDate date]])];
    NSDate *from = [fmt dateFromString:([fmt stringFromDate:self])];
    // 获取当前时间
    NSCalendar *calendar = [NSCalendar currentCalendar];
    NSCalendarUnit unit = NSCalendarUnitYear | NSCalendarUnitMonth | NSCalendarUnitDay;
    NSDateComponents  *comp  = [calendar components:unit fromDate:from toDate:now options:0];
    return comp.year == 0 && comp.month == 0 && comp.day == 1 ;
}
@end
```
