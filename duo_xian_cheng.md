# 多线程

## 基础知识
- 进程
	* 每一个程序在内存中都有一个进程
	* 每个进程之间是独立的，每个进程均运行在其专用且受保护的内存空间内
	* 一个进程可以有多个线程
- 线程
	* 一个进程要执行任务，必须有线程
	* 一个进程至少有一个线程
	* 线程是进程的基本执行单元
- 线程的执行方式
	- 串行，多个任务在线程内部执行顺序是串行的，类似于队列，先进先出
    - 也就是所同一时间只能有一个任务在执行
- 执行方式
	* 宏观是多个线程同时执行
	* 微观上是多个线程并发执行
	* 实际上CPU交替执行多个线程，这是每个线程的执行时间很短，而且CPU的切换速度比较快，所以给人得感觉是同时执行
- 优缺点
	* 优点
		- 适当提高程序的执行效率
		- 适当的提高资源的利用率，包括CPU和内存利用率
	* 缺点
		- 如果线程较多，会占用大量内存空间，反而会降低效率
		- 线程数量的增多，会加重CPU负担，频繁的进行CPU的调度，会把大量的时间浪费在线程的切换上
		- 线程的数量增多，程序的设计会更加复杂，主要是线程之间的通信，数据的访问

## IOS开发中多线程

- 主线程
	- 一个iOS程序运行后，默认会开启1条线程，称为“主线程”或“UI线程”
	- 作用
		- 显示和刷新界面
		- 处理UI事件（点击、滚动、拖拽等）
	- 注意事项
		- 耗时操作不能放在主线程中没，比如资源记载，文件下载，等比较耗时间的任务，不然会卡死界面
		- 可以将耗时操作放到子线程中，将操作结果返回给主线程
- IOS中得几种多线程实现方案
	- pThread
		- 一套通用的多线程API
		- 适用于Unix\Linux\Windows等系统
		- 跨平台\可移植
		- 使用难度大
		- C语言，手动管理线程生命周期

	- NSThread
		- 使用更加面向对象
简单易用，可直接操作线程对象
		- OC，手动管理线程生命周期
	- GCD
		- 旨在替代NSThread等线程技术，充分利用设备的多核
		- C，自动管理
	- NSOperation
		- 基于GCD（底层是GCD）
		- 比GCD多了一些更简单实用的功能
		- 使用更加面向对象
		- OC，自动管理
	- **其中GCD和NSOperation比较常用**

- pThread 的使用
了解即可

```objc
    // 多线程 pThread
    pthread_t thread;
    // 开启线程
    pthread_create(&thread, NULL, run, NULL);
    // 多线程 pThread
    pthread_t thread1;
    // 开启线程
    pthread_create(&thread1, NULL, run, NULL);

    void* run(void *para)
	{
    	for (int i = 0 ;i < 10000; i ++) {

        NSLog(@"run-%d----%@",i,[NSThread currentThread]);
    	}
   		return NULL;
	}
```

- NSThread 使用
	- 这种方式创建的线程，在执行完线程函数里的方法后就由系统销毁了

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
    // NSThread
    // 创建NSThread
    NSThread *thread1 = [[NSThread alloc] initWithTarget:self selector:@selector(threadRun:) object:@"鸟"];
    NSThread *thread2 = [[NSThread alloc] initWithTarget:self selector:@selector(threadRun:) object:@"蛋"];
    NSThread *thread3 = [[NSThread alloc] initWithTarget:self selector:@selector(threadRun:) object:@"碎了"];

    thread1.name = @"张三";
    thread2.name = @"李四";
    thread3.name = @"鸟";
    // 开启线程
    [thread1 start];
    [thread2 start];
    [thread3 start];

    // 这种方法创建的线程无法获得线程对象，由系统管理
    [NSThread detachNewThreadSelector:@selector(thread1:) toTarget:self withObject:@"狗蛋"];

}
- (void)threadRun:(id)obj
{
    for (int i = 0 ;i < 100; i ++) {

        NSLog(@"run-%d-%@---%@",i,obj,[NSThread currentThread]);
    }
}

```

## 创建子线程的其他方法

- **performSelectorInBackground**
- **performSelector**

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
    // 创建后台线程
    [self performSelectorInBackground:@selector(thread1:) withObject:@"控制器"];
    // 创建任务并添加到目标线程，可以是任意线程，子线程或者主线程
    [self performSelector:@selector(thread2:) onThread:[NSThread mainThread] withObject:@"Main" waitUntilDone:NO];

}

- (void)thread1:(NSString *)str
{
    for (int i = 0 ;i < 100; i ++) {

        NSLog(@"run-%@---%@",str,[NSThread currentThread]);
    }
}

- (void)thread2:(NSString *)str
{
    for (int i = 0 ;i < 100; i ++) {

        NSLog(@"run-%@---%@",str,[NSThread currentThread]);
    }
}
```

## 线程的睡眠（阻塞）和退出

```objc
- (void)thread1:(NSString *)str
{
    for (int i = 0 ;i < 100; i ++) {

        NSLog(@"run-%d-%@---%@",i,str,[NSThread currentThread]);
        if (i == 19) {
            sleep(2); // 当前线程睡眠2s
        }
        else if( i == 33 )
        {
            [NSThread sleepForTimeInterval:2]; // 睡眠2s
        }
        else if(i == 40)
        {
            // 获取当前时间延后2s后的时间
            NSDate *endDate = [NSDate dateWithTimeIntervalSinceNow:2];

            [NSThread sleepUntilDate:endDate]; // 根据日期睡眠线程
        }
        else if(i == 55)
        {
            NSLog(@"结束线程");
            [NSThread exit]; // 结束线程
        }
    }
}

```


## 线程同步 - 互斥锁

- 开启三个线程对同一个数据进行读写，就会出现问题，必须对读写数据进行处理，例如加锁
- 指令@synchronized()通过对一段代码的使用进行加锁。其他试图执行该段代码的线程都会被阻塞，直到加锁线程退出执行该段被保护的代码段，也就是说@synchronized()代码块中的最后一条语句已经被执行完毕的时候。
- @synchronized() 参数传入一个OC对象即可，self也可以

```objc
// 方法中对资源数的访问要加锁
- (void)saleTickets:(NSString *)str
{
    // 三个消费者线程
    while(1)
    {
        // 设置互斥锁
        @synchronized (self)
        {
            // 获取资源数
            NSInteger count = self.resourceCount;
            if (count > 0) {
                // 资源数减1
                count -- ;
                // 写入标记数据
                self.resourceCount = count;
                NSLog(@"%@消费了一个单位，还剩下%zd",[NSThread currentThread].name,count);
            }
            else
            {
                NSLog(@"资源全部使用完了");
                break;
            }
        }
    }

}
```

## 从网络下载图片并显示到UIImageView

### 首先开启一个后台线程

```objc
    // 开启以后台线程
    [self performSelectorInBackground:@selector(download2:) withObject:@"str"];
```

### 直接在子线程里显示图片

```objc
// 加载图片
- (void)download1:(NSString *)str
{
    NSLog(@"---%@",[NSThread currentThread]);
    CFTimeInterval begin = CFAbsoluteTimeGetCurrent(); // 获取当前时间
    NSLog(@"%f",begin);
    // 下载图片
    NSURL *url = [NSURL URLWithString:@"http://kuoo8.com/wall_up/hsf2288/201007/2010070921104598711.jpg"];
    NSData *data = [NSData dataWithContentsOfURL:url];
    CFTimeInterval end = CFAbsoluteTimeGetCurrent(); // 获取当前时间

    NSLog(@"%f",end);
    NSLog(@"---%f",end - begin); // 时间间隔
    // 直接在子线程里显示图片，这种方式不太好
    self.imageView.image = [UIImage imageWithData:data];
}
```
### 回到主线程显示图片

```objc
// 加载图片
- (void)download2:(NSString *)str
{
    NSLog(@"---%@",[NSThread currentThread]);
    CFTimeInterval begin = CFAbsoluteTimeGetCurrent(); // 获取当前时间
    NSLog(@"%f",begin);
    // 下载图片
    NSURL *url = [NSURL URLWithString:@"http://kuoo8.com/wall_up/hsf2288/201007/2010070921104598711.jpg"];
    NSData *data = [NSData dataWithContentsOfURL:url];
    CFTimeInterval end = CFAbsoluteTimeGetCurrent(); // 获取当前时间

    NSLog(@"%f",end);
    NSLog(@"---%f",end - begin); // 时间间隔
    UIImage *image = [UIImage imageWithData:data];

    // 回到主线程显示图片
    [self.imageView performSelector:@selector(setImage:) onThread:[NSThread mainThread] withObject:image waitUntilDone:NO];

}
```


## 获取从网络下载文件的时间间隔
- 方式1
  - NSDate 输出“ 2015-07-09 08:24:25 +0000”
- 方式2
  - NFTimeInterval 返回秒数 “458123091.828881”
- 使用方式也比较简单。

	- NSDate

	```objc
	NSDate *begin = [NSDate date]; // 获取当前日期
	 NSLog(@"%@",begin);
	// 下载图片
	NSURL *url = [NSURL URLWithString:@"http://kuoo8.com/wall_up/hsf2288/201007/2010070921104598711.jpg"];
	NSData *data = [NSData dataWithContentsOfURL:url];
	NSDate *end = [NSDate date];
	NSLog(@"%f",[end timeIntervalSinceDate:begin]);
	```

	- CFTimeInterval

	```objc
	CFTimeInterval begin = CFAbsoluteTimeGetCurrent(); // 获取当前时间
	NSLog(@"%f",begin);
	// 下载图片
	NSURL *url = [NSURL URLWithString:@"http://kuoo8.com/wall_up/hsf2288/201007/2010070921104598711.jpg"];
	NSData *data = [NSData dataWithContentsOfURL:url];
	CFTimeInterval end = CFAbsoluteTimeGetCurrent(); // 获取当前时间

	NSLog(@"%f",end);
	NSLog(@"---%f",end - begin); // 时间间隔

	```
## GCD

- GCD - Grand Central Dispatch
- GCD中任务和队列
	- 任务：执行什么操作
	- 队列：存放任务
- 使用方式
	- 定制自己的任务，添加到队列即可。线程会有GCD自动创建和销毁。


- **并发队列+异步任务**：创建多个线程，并发执行

```objc
 /*
 * 并发队列+异步任务：创建多个线程，并发执行
 */
- (void)concurrentAndAsync
{
    // 创建一个并发队列
    // 参数1：标识，一般用公司域名abc.com
    // 参数2：队列类型：串行和并行两种
    dispatch_queue_t queue = dispatch_queue_create("song.com", DISPATCH_QUEUE_CONCURRENT);
    // 异步方式创建一个任务，任务不会立即执行
    dispatch_async(queue, ^{
        NSLog(@"1--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"2--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"3--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"4--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"5--%@--",[NSThread currentThread]);
    });
    // 先将以上任务全部添加完毕，然后这个函数结束，再开始并发执行线程
    NSLog(@"dispatch_async--end -- %@--",[NSThread currentThread]);
    // 执行结果
    //    dispatch_async--end -- <NSThread: 0x7ff3b1400b80>{number = 1, name = main}--
//    2--<NSThread: 0x7ff3b140bc40>{number = 2, name = (null)}--
//    1--<NSThread: 0x7ff3b16672e0>{number = 5, name = (null)}--
//    3--<NSThread: 0x7ff3b1556b20>{number = 3, name = (null)}--
//    4--<NSThread: 0x7ff3b165c760>{number = 4, name = (null)}--
//    5--<NSThread: 0x7ff3b1413aa0>{number = 6, name = (null)}--

}
```


* **并发队列+同步任务**：不会开启新线程，在主线程中同步执行各个子线程，也就是逐一执行，并且是添加过任务后立即执行，之后才能添加下一个任务

```objc
 /*
 * 并发队列+同步任务：不会开启新线程，在父线程中同步执行各个子线程，也就是逐一执行，并且是添加过任务后立即执行，之后才能添加下一个任务
 */
- (void)concurrentAndSync
{
    // 创建一个并发队列
    // 参数1：标识，一般用公司域名abc.com
    // 参数2：队列类型：串行和并行两种
    dispatch_queue_t queue = dispatch_queue_create("song.com", DISPATCH_QUEUE_CONCURRENT);
    // 同步方式创建任务，任务会立即执行，执行完成后才会继续向下执行
    dispatch_sync(queue, ^{
        NSLog(@"1--%@--",[NSThread currentThread]);
    });
    dispatch_sync(queue, ^{
        NSLog(@"2--%@--",[NSThread currentThread]);
    });
    dispatch_sync(queue, ^{
        NSLog(@"3--%@--",[NSThread currentThread]);
    });
    dispatch_sync(queue, ^{
        NSLog(@"4--%@--",[NSThread currentThread]);
    });
    dispatch_sync(queue, ^{
        NSLog(@"5--%@--",[NSThread currentThread]);
    });
    // 先将以上任务逐个依次执行完毕后才会执行这句话，并且多有任务的执行都在主线程里完成
    NSLog(@"dispatch_sync--end -- %@--",[NSThread currentThread]);
    // 输出结果如下
    // 1--<NSThread: 0x7f9900711c90>{number = 1, name = main}--
    // 2--<NSThread: 0x7f9900711c90>{number = 1, name = main}--
    // 3--<NSThread: 0x7f9900711c90>{number = 1, name = main}--
    // ......
    // dispatch_sync--end -- <NSThread: 0x7f9900711c90>{number = 1, name = main}--
}
```

* **串行队列+同步任务**：不会开启新线程，在父线程中同步执行各个子线程，也就是逐一执行，并且是添加过任务后立即执行，之后才能添加下一个任务

```objc
/*
 * 串行队列+同步任务：不会开启新线程，在父线程中同步执行各个子线程，也就是逐一执行，并且是添加过任务后立即执行，之后才能添加下一个任务
 */
- (void)serialAndSync
{
    // 创建一个串行队列
    // 参数1：标识，一般用公司域名abc.com
    // 参数2：队列类型：串行和并行两种
    dispatch_queue_t queue = dispatch_queue_create("song.com", DISPATCH_QUEUE_SERIAL);
    // 同步方式创建任务，任务会立即执行，执行完成后才会继续向下执行
    dispatch_sync(queue, ^{
        NSLog(@"1--%@--",[NSThread currentThread]);
    });
    dispatch_sync(queue, ^{
        NSLog(@"2--%@--",[NSThread currentThread]);
    });
    dispatch_sync(queue, ^{
        NSLog(@"3--%@--",[NSThread currentThread]);
    });
    dispatch_sync(queue, ^{
        NSLog(@"4--%@--",[NSThread currentThread]);
    });
    dispatch_sync(queue, ^{
        NSLog(@"5--%@--",[NSThread currentThread]);
    });
    // 先将以上任务逐个依次执行完毕后才会执行这句话，并且多有任务的执行都在主线程里完成
    NSLog(@"dispatch_sync--end -- %@--",[NSThread currentThread]);
    // 1--<NSThread: 0x7fdc38f188d0>{number = 1, name = main}--
    // 2--<NSThread: 0x7fdc38f188d0>{number = 1, name = main}--
    // 3--<NSThread: 0x7fdc38f188d0>{number = 1, name = main}--
    // 4--<NSThread: 0x7fdc38f188d0>{number = 1, name = main}--
    // 5--<NSThread: 0x7fdc38f188d0>{number = 1, name = main}--
    // dispatch_sync--end -- <NSThread: 0x7fdc38f188d0>{number = 1, name = main}--
}
```

* **串行队列+异步任务**：创建新线程，但是只会创建一个新线程，所有的任务都是在这个子线程里执行，执行顺序按照添加任务 的先后顺序，并且不是立即执行，而是等整个方法

```objc
/*
 * 串行队列+异步任务：创建新线程，但是只会创建一个新线程，所有的任务都是在这个子线程里执行，执行顺序按照添加任务 的先后顺序，并且不是立即执行，而是等整个方法
 */
- (void)serialAndAsync
{
    // 创建一个串行队列
    // 参数1：标识，一般用公司域名abc.com
    // 参数2：队列类型：串行和并行两种
    dispatch_queue_t queue = dispatch_queue_create("song.com", DISPATCH_QUEUE_SERIAL);
    // 同步方式创建任务，任务会立即执行，执行完成后才会继续向下执行
    dispatch_async(queue, ^{
        NSLog(@"1--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"2--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"3--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"4--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"5--%@--",[NSThread currentThread]);
    });
    // 先将以上任务逐个依次执行完毕后才会执行这句话，并且多有任务的执行都在主线程里完成
    NSLog(@"dispatch_async--end -- %@--",[NSThread currentThread]);

    // dispatch_async--end -- <NSThread: 0x7f97f2611bf0>{number = 1, name = main}--
    // 1--<NSThread: 0x7f97f24461b0>{number = 2, name = (null)}--
    // 2--<NSThread: 0x7f97f24461b0>{number = 2, name = (null)}--
}

```

## GCD线程间通信

- 主队列 是GCD自带的一种特殊的串行队列
	- dispatch_get_main_queue()
- 全局并发队列
	- dispatch_get_global_queue(优先级,0)
全局并发队列的特性和手动创建的队列一样


```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    // 获取GCD自身的全局队列
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_async(queue, ^{
           NSLog(@"dispatch_get_global_queue--%@--",[NSThread currentThread]);
        // 下载图片
        NSURL *url = [NSURL URLWithString:@"http://kuoo8.com/wall_up/hsf2288/201007/2010070921104598711.jpg"];
        NSData *data = [NSData dataWithContentsOfURL:url];

        UIImage *image = [UIImage imageWithData:data];
        // 在主线程中显示图片
        dispatch_async(dispatch_get_main_queue(), ^{
            NSLog(@"dispatch_get_main_queue--%@--",[NSThread currentThread]);
            self.imageView.image = image;
        });
    });
}
```

* 主队列+异步任务：不会创建新线程，所有的任务都是在这个父线程里执行，执行顺序按照添加任务 的先后顺序，并且不是立即执行，而是等整个方法结束后依次执行


```objc
/*
 * 主队列+异步任务：不会创建新线程，所有的任务都是在这个父线程里执行，执行顺序按照添加任务 的先后顺序，并且不是立即执行，而是等整个方法结束后依次执行 */
- (void)mainAndAsync
{
    NSLog(@"dispatch_async--begin -- %@--",[NSThread currentThread]);
    // 创建一个串行队列
    // 参数1：标识，一般用公司域名abc.com
    // 参数2：队列类型：串行和并行两种
    dispatch_queue_t queue = dispatch_get_main_queue();;
    // 异步方式创建任务，任务不会立即执行
    dispatch_async(queue, ^{
        NSLog(@"1--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"2--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"3--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"4--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"5--%@--",[NSThread currentThread]);
    });
    // 先将以上任务逐个依次执行完毕后才会执行这句话，并且多有任务的执行都在主线程里完成
    NSLog(@"dispatch_get_main_queue--end -- %@--",[NSThread currentThread]);
}
```

* 主队列+同步任务：这个会产生问题,死锁，添加任务到主队列的任务要求立即执行，但是主队列是串行队列，当前任务要求执行完当前任务在执行新添加的任务。结果就是:两个任务互相等待，产生死锁

```objc
/*
 * 主队列+同步任务：这个会产生问题,死锁，添加任务到主队列的任务要求立即执行，但是主队列是串行队列，当前任务要求执行完当前任务在执行新添加的任务。结果就是:两个任务互相等待，产生死锁
 */
- (void)mainAndSync
{
    NSLog(@"dispatch_sync--begin -- %@--",[NSThread currentThread]);

    // 创建一个串行队列
    // 参数1：标识，一般用公司域名abc.com
    // 参数2：队列类型：串行和并行两种
    dispatch_queue_t queue = dispatch_get_main_queue();
    // 同步方式创建任务，任务会立即执行，执行完成后才会继续向下执行
    dispatch_sync(queue, ^{
        NSLog(@"1--%@--",[NSThread currentThread]);
    });
    dispatch_sync(queue, ^{
        NSLog(@"2--%@--",[NSThread currentThread]);
    });
    dispatch_sync(queue, ^{
        NSLog(@"3--%@--",[NSThread currentThread]);
    });
    dispatch_sync(queue, ^{
        NSLog(@"4--%@--",[NSThread currentThread]);
    });
    dispatch_sync(queue, ^{
        NSLog(@"5--%@--",[NSThread currentThread]);
    });
    // 先将以上任务逐个依次执行完毕后才会执行这句话，并且多有任务的执行都在主线程里完成
    NSLog(@"dispatch_get_main_queue--end -- %@--",[NSThread currentThread]);
}
```

* dispatch_barrier_async :截断线程执行，以上两个任务完成后才会执行这个任务，并在完成barrier后继续执行下面的任务
*  任务1，2的执行始终在任务3，4之前
*  只有在并发队列中才有阶段功能

```objc
/*
 * dispatch_barrier_async :截断线程执行，以上两个任务完成后才会执行这个任务，并在完成barrier后继续执行下面的任务
 */
- (void)barrier
{
    NSLog(@"dispatch_async--begin -- %@--",[NSThread currentThread]);

    // 创建一个串行队列
    // 参数1：标识，一般用公司域名abc.com
    // 参数2：队列类型：串行和并行两种
//    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_queue_t queue = dispatch_queue_create("song.com", DISPATCH_QUEUE_CONCURRENT);


    // 异步方式创建任务，任务不会立即执行
    dispatch_async(queue, ^{
        NSLog(@"1--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"2--%@--",[NSThread currentThread]);
    });
    // 截断线程执行，以上两个任务完成后才会执行这个任务，并在完成barrier后继续执行下面的任务
    dispatch_barrier_async(queue, ^{
        NSLog(@"dispatch_barrier_async--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"3--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"4--%@--",[NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"5--%@--",[NSThread currentThread]);
    });
    // 先将以上任务逐个依次执行完毕后才会执行这句话，并且多有任务的执行都在主线程里完成
    NSLog(@"dispatch_get_main_queue--end -- %@--",[NSThread currentThread]);
}
```

## 延时执行

```objc
- (void)delay
{
    // 延时执行
     NSLog(@"start--time:%@",[NSDate date]);
    // NSObject 方法
    [self performSelector:@selector(run) withObject:nil afterDelay:2];
    // NSTimer
    [NSTimer scheduledTimerWithTimeInterval:2 target:self selector:@selector(run) userInfo:nil repeats:NO];
    // GCD
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
          NSLog(@"run--time:%@",[NSDate date]);
    });

}
```

## 一次性代码
* 一次性代码内部默认就是线程安全的，不会出现多个线程同时访问内部代码

```objc
- (void)excuteOnce
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        NSLog(@"只执行一次---%@",[NSThread currentThread]);
    });
    NSLog(@"excuteOnce--%@",[NSThread currentThread]);
}
```
## 快速迭代

```objc
// 快速迭代,顺序不确定
- (void)apply
{
    dispatch_apply(10, dispatch_get_global_queue(0, 0), ^(size_t index) {
         NSLog(@"dispatch_apply--%ld--%@",index,[NSThread currentThread]);
    });
}
```
## 队列组

```objc
/*
 * 队列组:先执行async里的任务，最后执行notify任务
 */
- (void)groupAndAsync
{
    dispatch_group_t group = dispatch_group_create();

    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    // 先执行3个耗时操作
    dispatch_group_async(group, queue, ^{
        for (int i = 0 ; i < 1000; i ++) {
            NSLog(@"1--%@--",[NSThread currentThread]);
        }
    });

    dispatch_group_async(group, queue, ^{
        for (int i = 0 ; i < 1000; i ++) {
            NSLog(@"2--%@--",[NSThread currentThread]);
        }
    });
    dispatch_group_async(group, queue, ^{
        for (int i = 0 ; i < 1000; i ++) {
            NSLog(@"3--%@--",[NSThread currentThread]);
        }
    });
    // 等到以上任务完成后才会执行这个notify任务
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"main--%@--",[NSThread currentThread]);
    });
}
```
## 单例模式（普通）
- 保证一个类只有一个实例对象
- 在整个应用程序中，共享一份资源（这份资源只需要创建初始化1次）


```objc
@interface SLQPerson : NSObject

+ (instancetype)sharedInstance;

@end

@implementation SLQPerson

// 全局静态变量
static id _instance;

// alloc内部会调用allocWithZone方法，所以重载这个方法
+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    // 标记：用于判断任务是否完成
    static dispatch_once_t onceToken;
    // 一次性代码
    dispatch_once(&onceToken, ^{
        _instance = [super allocWithZone:zone];
    });
    return _instance;
}
// 提供1个类方法让外界访问唯一的实例
+ (instancetype)sharedInstance
{
    // 标记：用于判断任务是否完成
    static dispatch_once_t onceToken;
    // 一次性代码
    dispatch_once(&onceToken, ^{
        _instance = [[self alloc] init];
    });
    return _instance;
}
// copy对象
- (id)copyWithZone:(NSZone *zone)
{
    return _instance;
}
@end

```
- 不管是alloc 还是copy都是一个对象

```objc
SLQPerson *p1 = [[SLQPerson alloc] init];
SLQPerson *p2 = [SLQPerson sharedInstance];
SLQPerson *p3 = [SLQPerson sharedInstance];
SLQPerson *p4 = [p3 copy]; // 实现copyWithZone方法
// 实现allocWithZone方法即可
NSLog(@"%p--%p",p1,p2); // 0x7fce90612700--0x7fce90612700
NSLog(@"%p--%p",p3,p4); // 0x7fce90612700--0x7fce90612700
```
## 单例模式（宏）
- 宏的话，可以直接包含头文件进行使用

```objc
// 头文件
#define SLQInstanceInterface + (instancetype)sharedInstance;
// 实现文件
#define SLQInstanceImplementation \
static id _instance; \
 \
+ (instancetype)allocWithZone:(struct _NSZone *)zone \
{ \
    static dispatch_once_t onceToken; \
    dispatch_once(&onceToken, ^{ \
        _instance = [super allocWithZone:zone]; \
    }); \
    return _instance; \
} \
+ (instancetype)sharedInstance \
{ \
    static dispatch_once_t onceToken; \
    dispatch_once(&onceToken, ^{ \
        _instance = [[self alloc] init]; \
    }); \
    return _instance; \
}\
- (id)copyWithZone:(NSZone *)zone \
{\
    return _instance; \
}
```
## 单例模式宏（传入参数）

- 只需在宏定义时传入参数即可,##表示将参数拼接到后面
- 方法定义需要参数，实现也需要传入参数

```objc
#define SLQInstanceInterface(ClassName) + (instancetype)shared##ClassName;
```
```
+ (instancetype)shared##ClassName
```
- 使用时传入参数即可

```objc
@interface SLQPerson : NSObject

SLQInstanceInterface(Person)

@end

@implementation SLQPerson

SLQInstanceImplementation(Person)

@end

```
## 非GCD实现单例模式
- 如果要实现线程安全，那么必须进行加锁保护


- 如果这样写就会引起内存冲突

```objc
+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    if (_instance == nil) {
        sleep(2);
        _instance = [super allocWithZone:zone];
    }

    return _instance;
}
```
- 实现方式：加锁

```objc
// alloc内部会调用allocWithZone方法，所以重载这个方法
+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    @synchronized(self) // 访问加锁
    {
        if (_instance == nil) {
            sleep(2);
            _instance = [super allocWithZone:zone];
        }
    }
    return _instance;
}

// 提供1个类方法让外界访问唯一的实例
+ (instancetype)sharedInstance
{
    @synchronized(self)// 访问加锁
    {
        if (_instance == nil) {
            _instance = [[self alloc] init];
        }
    }
    return _instance;
}
```

## NSOperation
- NSOperation是一抽象类，只能使用它的子类NSBlockOperation和NSInvocationOperation，或者自定义NSOperation。

1、NSInvocationOperation
- 使用方法initWithTarget:进行初始化。
- 默认情况下，调用了start方法后并不会开一条新线程去执行操作，而是在当前线程同步执行操作。
- 只有将NSOperation放到一个NSOperationQueue中，才会异步执行操作。

```objc
    NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];
    [op start];
```
2、NSBlockOperation
- 使用blockOperationWithBlock添加任务，或者addExecutionBlock添加多个任务。
- 只要添加的任务数量大于1就会自动创建多个线程，异步执行。

```objc
 	NSBlockOperation *ope1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"0---%@",[NSThread currentThread]);
    }];

    [ope1 addExecutionBlock:^{
        NSLog(@"1---%@",[NSThread currentThread]);

    }];
    [ope1 addExecutionBlock:^{
        NSLog(@"2---%@",[NSThread currentThread]);

    }];
    [ope1 addExecutionBlock:^{
        NSLog(@"3---%@",[NSThread currentThread]);

    }];
    [ope1 start];
```
3、NSOperationQueue
- NSOperation可以调用start方法来执行任务，但默认是同步执行的。
- 如果将NSOperation添加到NSOperationQueue（操作队列）中，系统会自动异步执行NSOperation中的操作。

```objc
    // 添加到队列的任务会自动并发执行
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];

    NSBlockOperation *ope1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"1---%@",[NSThread currentThread]);
    }];
    NSBlockOperation *ope2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"2---%@",[NSThread currentThread]);
    }];

    NSBlockOperation *ope3 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"3---%@",[NSThread currentThread]);
    }];

    // 添加到队列会自动执行，不需要手动调用start方法
    [queue addOperation:ope1];
    [queue addOperation:ope2];
    [queue addOperation:ope3];
    [queue addOperationWithBlock:^{
        NSLog(@"4---%@",[NSThread currentThread]);
    }];
```
4、最大并发数
- 设置队列的最大并发数
- maxConcurrentOperationCount = 2；，同一时刻只会有两条线程在执行
- [queue setMaxConcurrentOperationCount:2]; // 这样设置
- 如果设置为1，就变成串行队列了

```objc
    // 添加到队列的任务会自动并发执行
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    // 最大并发数,同一时刻只会有两条线程在执行
    queue.maxConcurrentOperationCount = 2;

    // 添加到队列会自动执行，不需要手动调用start方法
    [queue addOperationWithBlock:^{
        for (int i = 0 ; i < 100; i ++) {

            NSLog(@"1---%@",[NSThread currentThread]);
        }
        [NSThread sleepForTimeInterval:0.01];
    }];
    [queue addOperationWithBlock:^{
        for (int i = 0 ; i < 100; i ++)
            NSLog(@"2---%@",[NSThread currentThread]);
        [NSThread sleepForTimeInterval:0.01];
    }];
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 100; i ++)
            NSLog(@"3---%@",[NSThread currentThread]);
        [NSThread sleepForTimeInterval:0.01];
    }];
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 100; i ++)
            NSLog(@"4---%@",[NSThread currentThread]);
        [NSThread sleepForTimeInterval:0.01];
    }];
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 100; i ++)
            NSLog(@"5---%@",[NSThread currentThread]);
        [NSThread sleepForTimeInterval:0.01];
    }];
```
5、线程队列的挂起、取消和恢复

- suspend ，挂起线程队列
- cancle

```objc
    if (self.queue.isSuspended) {
        // 唤醒线程队列
        self.queue.suspended = NO;
    }
    else
    {
         // 挂起线程队列
         self.queue.suspended = YES;
    }
```
- 取消线程的执行
- cancelAllOperations或cancel单个任务

```objc
    // 取消所有任务，立即取消线程中的所有任务
    [self.queue cancelAllOperations];
```
6、任务依赖  addDependency
- 多个线程之间可以建立依赖关系

```objc
    // 添加到队列的任务会自动并发执行
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    // 最大并发数,同一时刻只会有两条线程在执行
    queue.maxConcurrentOperationCount = 2;
    NSBlockOperation *ope1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"1---%@",[NSThread currentThread]);
    }];
    NSBlockOperation *ope2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"2---%@",[NSThread currentThread]);
    }];
    NSBlockOperation *ope3 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"3---%@",[NSThread currentThread]);
    }];
    NSBlockOperation *ope4 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"4---%@",[NSThread currentThread]);
    }];
    // 添加依赖：任务1和2的执行在3之前
    [ope3 addDependency:ope1];
    [ope3 addDependency:ope2];

    [queue addOperation:ope1];
    [queue addOperation:ope2];
    [queue addOperation:ope3];
    [queue addOperation:ope4];
```
7、线程通信
- 获取主队列mainQueue，将任务添加到主队列

```objc
    // 线程通信
    [[[NSOperationQueue alloc ]init] addOperationWithBlock:^{

        for (int i = 0 ; i < 1000; i ++)
            NSLog(@"1---%@",[NSThread currentThread]);

        // 回到主线程执行任务
        // 获取主队列
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
             NSLog(@"main---%@",[NSThread currentThread]);
        }];
    }];
```

```objc
    // 想在block内部访问这个变量要加__block,其实也就是改变了变量的作用域，使其在block内部也能被访问
    __block UIImage *image1 = nil;
    __block UIImage *image2 = nil;
    // 合成图片
    NSBlockOperation *downloadImage1 = [NSBlockOperation blockOperationWithBlock:^{
         // 下载图片1
        NSURL *url = [NSURL URLWithString:@"http://img2.zol.com.cn/product/117_940x705/630/ceY9kE6FBqYN2.jpg"];

        NSData *data = [NSData dataWithContentsOfURL:url];

        UIImage *image = [UIImage imageWithData:data];

        image1 = image;

    }];
    NSBlockOperation *downloadImage2 = [NSBlockOperation blockOperationWithBlock:^{
        // 下载图片2
        NSURL *url = [NSURL URLWithString:@"http://img.pconline.com.cn/../images/upload/upc/tx/photoblog/1403/29/c2/32592531_32592531_1396084944389_mthumb.jpg"];

        NSData *data = [NSData dataWithContentsOfURL:url];

        UIImage *image = [UIImage imageWithData:data];
        image2 = image;
    }];

    // 合成图片
    NSBlockOperation *combineImage = [NSBlockOperation blockOperationWithBlock:^{
        // 开启图形上下文
        UIGraphicsBeginImageContext(CGSizeMake(200, 200));
        // 绘制图片
        [image1 drawInRect:CGRectMake(0, 0, 100, 200)];
        [image2 drawInRect:CGRectMake(100, 0, 100, 200)];
        UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
        // 关闭图形上下文
        UIGraphicsEndImageContext();

        // 回到主线程刷新图片
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{

            self.imageView.image = image;
        }];
    }];

    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    // 添加依赖
    [combineImage addDependency:downloadImage1];
    [combineImage addDependency:downloadImage2];
    // 添加到队列
    [queue addOperation:downloadImage1];
    [queue addOperation:downloadImage2];
    [queue addOperation:combineImage];
```
- 下载网络图片显示在cell中


```objc

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    static NSString *ID = @"cell";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];
    // setup all cell with data
    // get item use parameter indexPath from Array:items
    SLQCellItem *item = self.items[indexPath.row];
    // init the cell use item
    cell.textLabel.text = item.name;
    cell.detailTextLabel.text = item.download;

    // 先从内存读取
    // 再从沙盒读取
    // 最后从网络下载

    // 从内存中读取数据
    UIImage *imageMemeory = self.../images[item.icon];

    if (imageMemeory) { // 如果内存中有
        cell.imageView.image = imageMemeory;
    }
    else // 内存中没有
    {
        // 读取沙盒中数据
        // 获取cache文件路径
        NSString *cachePath = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory,NSUserDomainMask , YES) firstObject];
        // 拼接字符串,将文件名和cache路径进行拼接,截取最后一个'/'后面的文件名
        NSString *filePath = [cachePath stringByAppendingPathComponent:[item.icon lastPathComponent]];
        // 读取沙盒中的图片信息
        NSData *dataCahe = [NSData dataWithContentsOfFile:filePath];
        UIImage *imageCache = [UIImage imageWithData:dataCahe];

        if (imageCache) { // 如果沙盒中图片存在
            // 加载到内存
            self.../images[item.icon] = imageCache;
            // 显示图片
            cell.imageView.image = imageCache;
        }
        else // 图片不在沙盒中，从网络下载
        {
            cell.imageView.image = [UIImage imageNamed:@"12"];
            // 这里可能会出现多个线程下载同一个图片,所以让每个任务添加到字典中
            // key 使用item.con
            NSOperation *operation = self.operations[item.icon];
            if(operation == nil) // 不存在下载item.icon的任务
            {
                // 开启线程下载图片
                operation = [NSBlockOperation blockOperationWithBlock:^{
                    // 在线程下载图片
                    NSURL *url = [NSURL URLWithString:item.icon];
                    NSData *data = [NSData dataWithContentsOfURL:url];
                    UIImage *imageDownload = [UIImage imageWithData:data];

                    if (imageDownload == nil)
                    {
                        // 移除字典中保存的路径
                        [self.operations removeObjectForKey:item.icon];
                        return ;
                    }
                    else
                    { // 如果图片下载成功
                        [NSThread sleepForTimeInterval:1.0];

                        // 回到主线程显示图片
                        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
                            // cell.imageView.image = imageDownload; // cell重用问题
                            [tableView reloadRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationNone];
                        }];

                        // 保存数据内存
                        self.../images[item.icon] = imageDownload;
                        // 保存数据到沙盒
                        [data writeToFile:filePath atomically:YES];
                        // 移除下载任务
                        [self.operations removeObjectForKey:item.icon];
                    }
                }];
                // 添加到任务队列
                [self.queue addOperation:operation];
                // 添加任务到字典
                self.operations[item.icon] = operation;
            }
        }
    }

    return cell;
}
```
