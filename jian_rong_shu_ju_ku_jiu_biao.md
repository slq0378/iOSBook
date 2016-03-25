# 兼容数据库旧表

- 问题描述
- 在版本1.6，1.7时使用了表T_xsxr，在版本1.8时使用了表名相同，结构不同的表，导致兼容性问题。
- 一些历史数据显示有问题，新数据插入表失败，

```objc
CREATE TABLE IF NOT EXISTS "T_xsxr" (
"keyid" text,
"prType" text,
"prLeadername" text,
"prNum" text,
"prMemo" text,
"prReportName" text,
"prReportUnit" text,
"prReportTime" text,
"prSendTime" text,
"address" text,
"longitude" text,
"latitude" text,
"fileAddressPhotos" text,
"isUpload" text
); // 1.6，1.7

CREATE TABLE IF NOT EXISTS "T_xsxr" (
"keyid" text,
"typename" text,
"leadername" text,
"peoplenumber" text,
"remark" text,
"reportname" text,
"reportdeptname" text,
"reporttime" text,
"reportaddress" text,
"longitude" text,
"latitude" text,
"filephotos" text,
"isUpload" text
);//1.8
```


- 解决方式，在版本1.9新建一个表,判断旧表是否存在，并且判断是旧表1，还是旧表2，分别读取旧表数据插入到新表然后删除旧表。

```objc
CREATE TABLE IF NOT EXISTS "T_jwbb" (
"keyid" text,
"typename" text,
"leadername" text,
"peoplenumber" text,
"remark" text,
"reportname" text,
"reportdeptname" text,
"reporttime" text,
"reportaddress" text,
"longitude" text,
"latitude" text,
"filephotos" text,
"isUpload" text
);//1.9
```


```objc
/// 兼容旧表
- (void)updateTable {
    //创建新表 T_jwbb
    [[SQLiteManager shareInstance] createTable:@"jwbb.sql"];
    __block BOOL existTable = NO;
    __block NSInteger T_xsxr_old = 0; // 0表示不存在这个表，1 表示旧表1，2表示旧表2
    __block NSInteger update = 0; // 0 不更新表，1 旧表1迁移至新表，2 旧表2迁移至新表
    [[SQLiteManager shareInstance].queue inDatabase:^(FMDatabase *db) {
         FMResultSet *set1 = [db executeQuery:@"SELECT COUNT(*) AS num FROM sqlite_master where type='table' and name='T_xsxr'"]; //
        
        if(set1.next)
        {
            if([[set1 objectForColumnName:@"num"] integerValue] == 0)  //not exist
            {
                existTable = NO;
            }
            else //existed
            {
                existTable = YES;
            }
        }
    }];
    
    [[SQLiteManager shareInstance].queue inDatabase:^(FMDatabase *db) {
        if (existTable) {
            FMResultSet *set2 = [db executeQuery:@"select * from T_xsxr"];
            NSString *str = [set2 columnNameForIndex:1];
            if ([str isEqualToString:@"prType"]) {
                T_xsxr_old = 1;
            }else if([str isEqualToString:@"typename"]) {
                T_xsxr_old = 2;
            }
        }
        else {
            T_xsxr_old = 0;// T_xsxr 表不存在
        }
    }];
    
    if (T_xsxr_old == 1) { // 旧表1存在
        /**/
        [[SQLiteManager shareInstance].queue inDatabase:^(FMDatabase *db) {
            BOOL isSucceed = [db executeUpdate:@"insert into T_jwbb(keyid, typename, leadername, peoplenumber, remark, reportname, reportdeptname, reporttime, reportaddress, longitude, latitude, filephotos, isUpload) select keyid, prType, prLeadername, prNum, prMemo, prReportName, prReportUnit, prReportTime, address, longitude, latitude, fileAddressPhotos, isUpload from T_xsxr"];
            if (isSucceed) {
                NSLog(@"旧表1存在，更新T_jwbb成功");
                update = 1;
            }else {
                NSLog(@"旧表1存在，更新T_jwbb失败");
            }
            [db close];
            
            if ([db open] && (update == 1 || update == 2)) {
                
                BOOL isSucceed1 =  [db executeUpdate:@"drop table T_xsxr;"];
                if (isSucceed1) {
                    NSLog(@"删除T_xsxr成功");
                }
                else {
                    NSLog(@"删除T_xsxr失败");
                }
            }
        }];
    }else if(T_xsxr_old == 2){ // 旧表2存在
        /**/
        [[SQLiteManager shareInstance].queue inDatabase:^(FMDatabase *db){
           BOOL isSucceed = [db executeUpdate:@"insert into T_jwbb(keyid, typename, leadername, peoplenumber, remark, reportname, reportdeptname, reporttime, reportaddress, longitude, latitude, filephotos, isUpload) select keyid, typename, leadername, peoplenumber, remark, reportname, reportdeptname, reporttime, reportaddress, longitude, latitude, filephotos, isUpload from T_xsxr"];
            if (isSucceed) {
                update = 2;
                NSLog(@"旧表2存在，更新T_jwbb成功");
            }else {
                NSLog(@"旧表2存在，更新T_jwbb失败");
            }
            [db close];
            
            if ([db open] && (update == 1 || update == 2)) {
                
                BOOL isSucceed1 =  [db executeUpdate:@"drop table T_xsxr;"];
                if (isSucceed1) {
                    NSLog(@"删除T_xsxr成功");
                }
                else {
                    NSLog(@"删除T_xsxr失败");
                }
            }
        }];
    }else { // 不存在旧表
        NSLog(@"不存在旧表");
    }
}
```

