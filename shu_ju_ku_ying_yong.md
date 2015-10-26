# SQLite

## SLQLite简介

- 什么是SQLite
	- SQLite是一款轻型的嵌入式数据库
	- 它占用资源非常的低，在嵌入式设备中，可能只需要几百K的内存就够了
	- 它的处理速度比Mysql、PostgreSQL这两款著名的数据库都还快
- 什么是数据库
	- 数据库（Database）是按照数据结构来组织、存储和管理数据的仓库
	- 数据库可以分为2大种类
	- 关系型数据库（主流）
	- 对象型数据库
- 常用关系型数据库
	- PC端：Oracle、MySQL、SQL Server、Access、DB2、Sybase
	- 嵌入式\移动客户端：SQLite


## Navicat 界面化管理工具

- 使用方式参考官方文档就好
- <http://www.navicat.com.cn/products/navicat-for-sqlite>

## SQL语句

- 数据定义语句（DDL：Data Definition Language）
	- 包括create和drop等操作
	- 在数据库中创建新表或删除表（create table或 drop table）
- 数据操作语句（DML：Data Manipulation Language）
	- 包括insert、update、delete等操作
	- 上面的3种操作分别用于添加、修改、删除表中的数据
- 数据查询语句（DQL：Data Query Language）
	- 可以用于查询获得表中的数据
	- 关键字select是DQL（也是所有SQL）用得最多的操作
	- 其他DQL常用的关键字有where，order by，group by和having

## SQLite3使用

- 在iOS中使用SQLite3，首先要添加库文件libsqlite3.dylib和导入主头文件
- 导入sqlite3.h


- 打开数据库

	
```
	int sqlite3_open(
	    const char *filename,   // 数据库的文件路径
	    sqlite3 **ppDb          // 数据库实例
	);
```


- 执行任何SQL语句



```
	int sqlite3_exec(
	    sqlite3*,                                  // 一个打开的数据库实例
	    const char *sql,                           // 需要执行的SQL语句
	    int (*callback)(void*,int,char**,char**),  // SQL语句执行完毕后的回调
	    void *,                                    // 回调函数的第1个参数
	    char **errmsg                              // 错误信息
	);
```

- 检查SQL语句的合法性（查询前的准备）



```
int sqlite3_prepare_v2(
    sqlite3 *db,            // 数据库实例
    const char *zSql,       // 需要检查的SQL语句
    int nByte,              // SQL语句的最大字节长度
    sqlite3_stmt **ppStmt,  // sqlite3_stmt实例，用来获得数据库数据
    const char **pzTail
);
```

- 查询一行数据



```
	int sqlite3_step(sqlite3_stmt*); // 如果查询到一行数据，就会返回SQLITE_ROW
```

- 利用stmt获得某一字段的值（字段的下标从0开始）



```
	double sqlite3_column_double(sqlite3_stmt*, int iCol);  // 浮点数据
	int sqlite3_column_int(sqlite3_stmt*, int iCol); // 整型数据
	sqlite3_int64 sqlite3_column_int64(sqlite3_stmt*, int iCol); // 长整型数据
	const void *sqlite3_column_blob(sqlite3_stmt*, int iCol); // 二进制文本数据
	const unsigned char *sqlite3_column_text(sqlite3_stmt*, int iCol);  // 字符串数据
```

- 代码演示


```
	/*简单约束*/
	CREATE TABLE IF NOT EXISTS t_student(id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, age INTEGER);
	CREATE TABLE IF NOT EXISTS t_student(id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL, age INTEGER NOT NULL);
	CREATE TABLE IF NOT EXISTS t_student(id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT UNIQUE, age INTEGER);
	CREATE TABLE IF NOT EXISTS t_student(id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, age INTEGER DEFAULT 1);

	/*分页*/
	SELECT * FROM t_student ORDER BY id ASC LIMIT 30, 10;

	/*排序*/
	SELECT * FROM t_student WHERE score > 50 ORDER BY age DESC;
	SELECT * FROM t_student WHERE score < 50 ORDER BY age ASC , score DESC;

	/*计量*/
	SELECT COUNT(*) FROM t_student WHERE age > 50;

	/*别名*/
	SELECT name as myName, age as myAge, score as myScore FROM t_student;
	SELECT name myName, age myAge, score myScore FROM t_student;
	SELECT s.name myName, s.age myAge, s.score myScore FROM t_student s WHERE s.age > 50;

	/*查询*/
	SELECT name, age, score FROM t_student;
	SELECT * FROM t_student;

	/*修改指定数据*/
	UPDATE t_student SET name = 'MM' WHERE age = 10;
	UPDATE t_student SET name = 'WW' WHERE age is 7;
	UPDATE t_student SET name = 'XXOO' WHERE age < 20;
	UPDATE t_student SET name = 'NNMM' WHERE age < 50 and score > 10;

	/*删除数据*/
	DELETE FROM t_student;

	/*更新数据*/
	UPDATE t_student SET name = 'LNJ';

	/*插入数据*/

	 INSERT INTO t_student(age, score, name) VALUES ('28', 100, 'jonathan');
	 INSERT INTO t_student(name, age) VALUES ('lee', '28');
	 INSERT INTO t_student(score) VALUES (100);

	/*插入数据*/
	INSERT INTO t_student(name, age, score) VALUES ('lee', '28', 100);

	/*添加主键*/
	CREATE TABLE IF NOT EXISTS t_student (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, age INTEGER, score REAL);
	/*添加主键*/
	CREATE TABLE IF NOT EXISTS t_student (id INTEGER, name TEXT, age INTEGER, score REAL, PRIMARY KEY(id));

	/*删除表*/
	DROP TABLE IF EXISTS t_student;

	/*创建表*/
	CREATE TABLE IF NOT EXISTS t_student(id INTEGER , name TEXT, age , score REAL);
```

## 使用示例

- 初始化数据库


```
	// static的作用：能保证_db这个变量只被本文件直接访问
	static sqlite3 *_db;

	+ (void)initialize
	{
	    // 0.获得沙盒中的数据库文件名
	    NSString *filename = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:@"student.sqlite"];

	    // 1.创建(打开)数据库（如果数据库文件不存在，会自动创建）
	    int result = sqlite3_open(filename.UTF8String, &_db);
	    if (result == SQLITE_OK) {
	        NSLog(@"成功打开数据库");

	        // 2.创表
	        const char *sql = "create table if not exists t_student (id integer primary key autoincrement, name text, age integer);";
	        char *errorMesg = NULL;
	        int result = sqlite3_exec(_db, sql, NULL, NULL, &errorMesg);
	        if (result == SQLITE_OK) {
	            NSLog(@"成功创建t_student表");
	        } else {
	            NSLog(@"创建t_student表失败:%s", errorMesg);
	        }
	    } else {
	        NSLog(@"打开数据库失败");
	    }
	}
```

- 增删改查


```
	+ (BOOL)addStudent:(IWStudent *)student
	{
	    NSString *sql = [NSString stringWithFormat:@"insert into t_student (name, age) values('%@', %d);", student.name, student.age];

	    char *errorMesg = NULL;
	    int result = sqlite3_exec(_db, sql.UTF8String, NULL, NULL, &errorMesg);

	    return result == SQLITE_OK;
	}
	+ (NSArray *)students
	{
	    // 0.定义数组
	    NSMutableArray *students = nil;

	    // 1.定义sql语句
	    const char *sql = "select id, name, age from t_student;";

	    // 2.定义一个stmt存放结果集
	    sqlite3_stmt *stmt = NULL;

	    // 3.检测SQL语句的合法性
	    int result = sqlite3_prepare_v2(_db, sql, -1, &stmt, NULL);
	    if (result == SQLITE_OK) {
	        NSLog(@"查询语句是合法的");
	        students = [NSMutableArray array];

	        // 4.执行SQL语句，从结果集中取出数据
	        while (sqlite3_step(stmt) == SQLITE_ROW) { // 真的查询到一行数据
	            // 获得这行对应的数据

	            IWStudent *student = [[IWStudent alloc] init];

	            // 获得第0列的id
	            student.ID = sqlite3_column_int(stmt, 0);

	            // 获得第1列的name
	            const unsigned char *sname = sqlite3_column_text(stmt, 1);
	            student.name = [NSString stringWithUTF8String:(const char *)sname];

	            // 获得第2列的age
	            student.age = sqlite3_column_int(stmt, 2);

	            // 添加到数组
	            [students addObject:student];
	        }
	    } else {
	        NSLog(@"查询语句非合法");
	    }

	    return students;
	}
	+ (NSArray *)studentsWithCondition:(NSString *)condition
	{
	    // 0.定义数组
	    NSMutableArray *students = nil;

	    // 1.定义sql语句
	    const char *sql = "select id, name, age from t_student where name like ?;";

	    // 2.定义一个stmt存放结果集
	    sqlite3_stmt *stmt = NULL;

	    // 3.检测SQL语句的合法性
	    int result = sqlite3_prepare_v2(_db, sql, -1, &stmt, NULL);
	    if (result == SQLITE_OK) {
	        NSLog(@"查询语句是合法的");
	        students = [NSMutableArray array];

	        // 填补占位符的内容
	        NSString *newCondition = [NSString stringWithFormat:@"%%%@%%", condition];
	//        NSLog(@"%@", newCondition);
	        sqlite3_bind_text(stmt, 1, newCondition.UTF8String, -1, NULL);

	        // 4.执行SQL语句，从结果集中取出数据
	        while (sqlite3_step(stmt) == SQLITE_ROW) { // 真的查询到一行数据
	            // 获得这行对应的数据

	            IWStudent *student = [[IWStudent alloc] init];

	            // 获得第0列的id
	            student.ID = sqlite3_column_int(stmt, 0);

	            // 获得第1列的name
	            const unsigned char *sname = sqlite3_column_text(stmt, 1);
	            student.name = [NSString stringWithUTF8String:(const char *)sname];

	            // 获得第2列的age
	            student.age = sqlite3_column_int(stmt, 2);

	            // 添加到数组
	            [students addObject:student];
	        }
	    } else {
	        NSLog(@"查询语句非合法");
	    }

	    return students;
	}
```

## FMDB

- FMDB是iOS平台的SQLite数据库框架,FMDB以OC的方式封装了SQLite的C语言API
- FMDB有三个主要的类
- FMDatabase
	- 一个FMDatabase对象就代表一个单独的SQLite数据库
	- 用来执行SQL语句
- FMResultSet
	- 使用FMDatabase执行查询后的结果集
- FMDatabaseQueue
	- 用于在多线程中执行多个查询或更新，它是线程安全的

### 打开数据库

- 通过指定SQLite数据库文件路径来创建FMDatabase对象

```
	FMDatabase *db = [FMDatabase databaseWithPath:path];
	if (![db open]) {
	    NSLog(@"数据库打开失败！");
	}
```

- 文件路径有三种情况
	- 具体文件路径
	- 如果不存在会自动创建
- 空字符串@""
	- 会在临时目录创建一个空的数据库
	- 当FMDatabase连接关闭时，数据库文件也被删除
- nil
	- 会创建一个内存中临时数据库，当FMDatabase连接关闭时，数据库会被销毁

### 执行更新操作

- 在FMDB中，除查询以外的所有操作，都称为“更新”
	- create、drop、insert、update、delete等

- 使用executeUpdate:方法执行更新

```
	- (BOOL)executeUpdate:(NSString*)sql, ...
	- (BOOL)executeUpdateWithFormat:(NSString*)format, ...
	- (BOOL)executeUpdate:(NSString*)sql withArgumentsInArray:(NSArray *)arguments
```

- 示例
- `[db executeUpdate:@"UPDATE t_student SET age = ? WHERE name = ?;", @20, @"Jack"]`

### 执行查询


- 查询方法


```
	- (FMResultSet *)executeQuery:(NSString*)sql, ...
	- (FMResultSet *)executeQueryWithFormat:(NSString*)format, ...
	- (FMResultSet *)executeQuery:(NSString *)sql withArgumentsInArray:(NSArray *)arguments
```


- 示例


```
	// 查询数据
	FMResultSet *rs = [db executeQuery:@"SELECT * FROM t_student"];
	
	// 遍历结果集
	while ([rs next]) {
	    NSString *name = [rs stringForColumn:@"name"];
	    int age = [rs intForColumn:@"age"];
	    double score = [rs doubleForColumn:@"score"];
	}
```

## 多线程操作

- 如果要进行海量操作的话，就要用到子线程来加快速度
- 但是数据库操作海量数据的话，比如说插入100000条数据的话，可以开启多个子线程来操作，但是必须使用同步线程，让多个线程逐一执行，而不致于引起数据库数据混乱。
- `FMDatabaseQueue`使用这个类


```
	//FMDatabaseQueue的创建
	FMDatabaseQueue *queue = [FMDatabaseQueue databaseQueueWithPath:path];
```

- 基本操作


```
	[queue inDatabase:^(FMDatabase *db) {
	    [db executeUpdate:@"INSERT INTO t_student(name) VALUES (?)", @"Jack"];
	    [db executeUpdate:@"INSERT INTO t_student(name) VALUES (?)", @"Rose"];
	    [db executeUpdate:@"INSERT INTO t_student(name) VALUES (?)", @"Jim"];
	    
	    FMResultSet *rs = [db executeQuery:@"select * from t_student"];
	    while ([rs next]) {
	        // …
	    }
	}];
```

- 使用事务


```
	[queue inTransaction:^(FMDatabase *db, BOOL *rollback) {
	    [db executeUpdate:@"INSERT INTO t_student(name) VALUES (?)", @"Jack"];
	    [db executeUpdate:@"INSERT INTO t_student(name) VALUES (?)", @"Rose"];
	    [db executeUpdate:@"INSERT INTO t_student(name) VALUES (?)", @"Jim"];
	    FMResultSet *rs = [db executeQuery:@"select * from t_student"];
	    while ([rs next]) {
	        // …
	    }
	}];
```

- 事务回滚
	- `*rollback = YES;`

