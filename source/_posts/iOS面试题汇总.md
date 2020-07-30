---
title: iOS 面试题汇总
date: 2019-02-25 16:25:00
tags: 代码库
---

**1. 简单介绍下NSURLConnection类及`+ sendSynchronousRequest:returningResponse:error:`与`– initWithRequest:delegate:`两个方法的区别?**
> 答: NSURLConnection主要用于网络访问，其中+ sendSynchronousRequest:returningResponse:error:是同步访问数据，即当前线程会阻塞，并等待request的返回的response，而– initWithRequest:delegate:使用的是异步加载，当其完成网络访问后，会通过delegate回到主线程，并其委托的对象。

**2. 在项目什么时候选择使用GCD，什么时候选择NSOperation**
> 答: 项目中使用NSOperation的优点是NSOperation是对线程的高度抽象，在项目中使用它，会使项目的程序结构更好，子类化NSOperation的设计思路，是具有面向对象的优点(复用、封装)，使得实现是多线程支持，而接口简单，建议在复杂项目中使用。
项目中使用GCD的优点是GCD本身非常简单、易用，对于不复杂的多线程操作，会节省代码量，而Block参数的使用，会是代码更为易读，建议在简单项目中使用。

**3. ViewController的didReceiveMemoryWarning怎么被调用**
> 答:[supper didReceiveMemoryWarning];

**4. 写一个setter方法用于完成`@property(nonatomic, retain) NSString *name`,写一个setter方法用于完成`@property(nonatomic, copy) NSString *name`**
```
- (void)setName:(NSString *)str{
    [str retain];
    [_name release];
    _name = str;
}
- (void)setName:(NSString *)str{
    id t = [str copy];
    [_name release];
    _name = t;
}
```
**5. 对于语句`NSString *obj = [[NSData alloc] init];` obj在编译时和运行时分别时什么类型的对象?**
> 答： 编译时是NSString的类型;运行时是NSData类型的对象

**6. Object C中创建线程的方法是什么?如果在主线程中执行代码，方法是什么?如果想延时执行代码、方法又是什么?**
> 答：线程创建有三种方法：使用NSThread创建、使用GCD的dispatch、使用子类化的NSOperation,然后将其加入NSOperationQueue;在主线程执行代码，方法是performSelectorOnMainThread，如果想延时执行代码可以用performSelector:onThread:withObject:waitUntilDone:

**7. 浅复制和深复制的区别?**
> 答：浅层复制：只复制指向对象的指针，而不复制引用对象本身。
深层复制：复制引用对象本身。

**8. PerformSelecter**
> 当调用 NSObject 的`performSelecter:afterDelay:`后，实际上其内部会创建一个 Timer 并添加到当前线程的 RunLoop 中。所以如果当前线程没有 RunLoop，则这个方法会失效。
当调用`performSelector:onThread:`时，实际上其会创建一个 Timer 加到对应的线程去，同样的，如果对应线程没有 RunLoop 该方法也会失效。

**9. 优化你是从哪几方面着手？**
> 一、首页启动速度
 启动过程中做的事情越少越好（尽可能将多个接口合并）
 不在UI线程上作耗时的操作（数据的处理在子线程进行，处理完通知主线程刷新节目）
在合适的时机开始后台任务（例如在用户指引节目就可以开始准备加载的数据）
二、页面浏览速度
json的处理（iOS 自带的NSJSONSerialization，Jsonkit，SBJson）
数据的分页（后端数据多的话，就要分页返回，例如网易新闻，或者 微博记录）
数据压缩（大数据也可以压缩返回，减少流量，加快反应速度）
内容缓存（例如网易新闻的最新新闻列表都是要缓存到本地，从本地加载，可以缓存到内存，或者数据库，根据情况而定）
延时加载tab（比如app有5个tab，可以先加载第一个要显示的tab，其他的在显示时候加载，按需加载）
算法的优化（核心算法的优化，例如有些app 有个 联系人姓名用汉语拼音的首字母排序）
三、操作流畅度优化
Tableview 优化（tableview cell的加载优化）
ViewController加载优化（不同view之间的跳转，可以提前准备好数据）
四、数据库的优化
数据库设计上面的重构
查询语句的优化
分库分表（数据太多的时候，可以分不同的表或者库）
五、服务器端和客户端的交互优化
客户端尽量减少请求
服务端尽量做多的逻辑处理
服务器端和客户端采取推拉结合的方式（可以利用一些同步机制）
通信协议的优化（减少报文的大小）
电量使用优化（尽量不要使用后台运行）
六、非技术性能优化
产品设计的逻辑性（产品的设计一定要符合逻辑，或者逻辑尽量简单，否则会让程序员抓狂，有时候用了好大力气，才可以完成一个小小的逻辑设计问题）
界面交互的规范（每个模块的界面的交互尽量统一，符合操作习惯）
代码规范（这个可以隐形带来app 性能的提高，比如 用if else 还是switch ，或者是用！还是 ＝＝）
code review（坚持code Review 持续重构代码。减少代码的逻辑复杂度）

**10. 什么情况使用 weak 关键字，相比 assign 有什么不同？**
> 1.在ARC中,在有可能出现循环引用的时候,往往要通过让其中一端使用 weak 来解决,比如: delegate 代理属性。
2.自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 weak,如自定义 IBOutlet 控件属性一般也使用 weak；当然，也可以使用strong。
IBOutlet连出来的视图属性为什么可以被设置成weak?
答：因为父控件的subViews数组已经对它有一个强引用。
不同点
assign 可以用非 OC 对象，而 weak 必须用于 OC 对象。
weak 表明该属性定义了一种“非拥有关系”。在属性所指的对象销毁时，属性值会自动清空(nil)

**11. 用@property声明的 NSString / NSArray / NSDictionary 经常使用 copy 关键字，为什么？如果改用strong关键字，可能造成什么问题？**
> 答：用 @property 声明 NSString、NSArray、NSDictionary 经常使用 copy 关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary，他们之间可能进行赋值操作（就是把可变的赋值给不可变的），为确保对象中的字符串值不会无意间变动，应该在设置新属性值时拷贝一份。
1.因为父类指针可以指向子类对象,使用 copy 的目的是为了让本对象的属性不受外界影响,使用 copy 无论给我传入是一个可变对象还是不可对象,我本身持有的就是一个不可变的副本。
2.如果我们使用是 strong ,那么这个属性就有可能指向一个可变对象,如果这个可变对象在外部被修改了,那么会影响该属性。
总结：使用copy的目的是，防止把可变类型的对象赋值给不可变类型的对象时，可变类型对象的值发送变化会无意间篡改不可变类型对象原来的值。

**12. runtime如何实现weak变量的自动置nil？**
> runtime对注册的类，会进行布局，会将 weak 对象放入一个 hash 表中。用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会调用对象的 dealloc 方法，假设 weak 指向的对象内存地址是a，那么就会以a为key，在这个 weak hash表中搜索，找到所有以a为key的 weak 对象，从而设置为nil

**13. runloop是什么／runloop的概念？**
> runloop是线程相关的基础框架的一部分。一个runloop就是一个事件处理的循环，用来不停的调度工作以及处理输入事件。其实内部就是do－while循环，这个循环内部不断地处理各种任务（比如Source，Timer，Observer）。使用runloop的目的是让你的线程在有工作的时候忙于工作，而没工作的时候处于休眠状态。

**14. UITableViewCell上有个UILabel，显示NSTimer实现的秒表时间，手指滚动cell过程中，label是否刷新，为什么？**
> 这是否刷新取决于timer加入到Run Loop中的Mode是什么。Mode主要是用来指定事件在运行循环中的优先级的，分为
> * NSDefaultRunLoopMode（kCFRunLoopDefaultMode）：默认，空闲状态
> * UITrackingRunLoopMode：ScrollView滑动时会切换到该Mode
> * UIInitializationRunLoopMode：run loop启动时，会切换到该mode
> * NSRunLoopCommonModes（kCFRunLoopCommonModes）：Mode集合
苹果公开提供的Mode有两个
> * NSDefaultRunLoopMode（kCFRunLoopDefaultMode）
> * NSRunLoopCommonModes（kCFRunLoopCommonModes）
在编程中：如果我们把一个NSTimer对象以NSDefaultRunLoopMode（kCFRunLoopDefaultMode）添加到主运行循环中的时候, ScrollView滚动过程中会因为mode的切换，而导致NSTimer将不再被调度。当我们滚动的时候，也希望不调度，那就应该使用默认模式。但是，如果希望在滚动时，定时器也要回调，那就应该使用common mode。

**15. NStimer准吗？谈谈你的看法？如果不准该怎样实现一个精确的NSTimer?**
> 不准；不准的原因如下
1、NSTimer加在main runloop中，模式是NSDefaultRunLoopMode，main负责所有主线程事件，例如UI界面的操作，复杂的运算，这样在同一个runloop中timer就会产生阻塞。
2、模式的改变。主线程的 RunLoop 里有两个预置的 Mode：kCFRunLoopDefaultMode 和 UITrackingRunLoopMode。
当你创建一个 Timer 并加到 DefaultMode 时，Timer 会得到重复回调，但此时滑动一个ScrollView时，RunLoop 会将 mode 切换为 TrackingRunLoopMode，这时 Timer 就不会被回调，并且也不会影响到滑动操作。所以就会影响到NSTimer不准的情况。
PS:DefaultMode 是 App 平时所处的状态，rackingRunLoopMode 是追踪 ScrollView 滑动时的状态。
方法：
1、在主线程中进行NSTimer操作，但是将NSTimer实例加到main runloop的特定mode（模式）中。避免被复杂运算操作或者UI界面刷新所干扰self.timer = [NSTimer timerWithTimeInterval:1 target:self selector:@selector(showTime) userInfo:nil repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSRunLoopCommonModes];
2、在子线程中进行NSTimer的操作，再在主线程中修改UI界面显示操作结果
-(void)timerMethod2 {
  NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(newThread) object:nil];
  [thread start];
}
-(void)newThread{
  @autoreleasepool{
    [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(showTime) userInfo:nil repeats:YES];
    [[NSRunLoop currentRunLoop] run];
   }
}

**16. NSOperation 相比于 GCD 有哪些优势？**
> GCD是基于c的底层api，NSOperation属于object-c类。ios 首先引入的是NSOperation，IOS4之后引入了GCD和NSOperationQueue并且其内部是用gcd实现的。
相对于GCD：
1、NSOperation拥有更多的函数可用，具体查看api。
2、在NSOperationQueue中，可以建立各个NSOperation之间的依赖关系。
3、有kvo可以监测operation是否正在执行（isExecuted）、是否结束（isFinished），是否取消（isCanceld）。
4、NSOperationQueue可以方便的管理并发、NSOperation之间的优先级。
GCD主要与block结合使用。代码简洁高效。
GCD也可以实现复杂的多线程应用，主要是建立个个线程时间的依赖关系这类的情况，但是需要自己实现相比NSOperation要复杂。
具体使用哪个，依需求而定。 从个人使用的感觉来看，比较合适的用法是：除了依赖关系尽量使用GCD，因为苹果专门为GCD做了性能上面的优化。

**17. 如何访问并修改一个类的私有属性?**
> 有两种方法可以访问私有属性,一种是通过KVC获取,一种是通过runtime访问并修改私有属性。

**18. 如何捕获异常？**
```
1. 在app启动时(didFinishLaunchingWithOptions)，添加一个异常捕获的监听
NSSetUncaughtExceptionHandler(&UncaughtExceptionHandler);
2. 实现捕获异常日志并保存到本地的方法
void UncaughtExceptionHandler(NSException *exception){
    //异常日志获取
    NSArray  *excpArr = [exception callStackSymbols];
    NSString *reason = [exception reason];
    NSString *name = [exception name];
    NSString *excpCnt = [NSString stringWithFormat:@"exceptionType: %@ \n reason: %@ \n stackSymbols: %@",name,reason,excpArr];
    //日常日志保存（可以将此功能单独提炼到一个方法中）
    NSArray  *dirArr  = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *dirPath = dirArr[0];
    NSString *logDir = [dirPath stringByAppendingString:@"/CrashLog"];

    BOOL isExistLogDir = YES;
    NSFileManager *fileManager = [NSFileManager defaultManager];
    if (![fileManager fileExistsAtPath:logDir]) {
        isExistLogDir = [fileManager createDirectoryAtPath:logDir withIntermediateDirectories:YES attributes:nil error:nil];
    }
    if (isExistLogDir) {
        //此处可扩展
        NSString *logPath = [logDir stringByAppendingString:@"/crashLog.txt"];
        [excpCnt writeToFile:logPath atomically:YES encoding:NSUTF8StringEncoding error:nil];
    }
}
```
**19. Object-c的类可以多重继承么?可以实现多个接口么?Category是什么?重写一个类的方式用继承好还是分类好?为什么?**
> 答：Object-c的类不可以多重继承;可以实现多个接口，通过实现多个接口可以完成C++的多重继承;Category是类别，一般情况用分类好，用Category去重写类的方法，仅对本Category有效，不会影响到其他类与原有类的关系。

**20. Category(分类)，Extension(扩展)和继承的区别**
> 答：1.分类
category原则上只能在现有类基础上添加新的方法（能添加属性的原因只是通过runtime解决无setter/getter的问题而已），类别中的方法没被实现编译器是不会有任何警告的，这是因为类别是在运行时添加到类中的
2.扩展
iOS中的extension就是匿名的分类，只有头文件没有实现文件。类扩展不仅可以增加方法，还可以增加实例变量（或者属性），只是该实例变量默认是@private类型的（使用范围只能在自身类，而不是子类或其他地方），类扩展中声明的方法没被实现，编译器会报警，这是因为类扩展是在编译阶段被添加到类中的
3.继承
在iOS中继承是单继承，既只能有一个父类。在继承中，子类可以使用父类的方法和变量，当子类想对本类或者父类的变量进行初始化，那么需要重写init()方法 。父类也可以访问子类的方法和成员变量

**21. 简述内存分区情况**
> 1).代码区：存放函数二进制代码
   2).数据区：系统运行时申请内存并初始化，系统退出时由系统释放。存放全局变量、静态变量、常量
   3).堆区：通过malloc等函数或new等操作符动态申请得到，需程序员手动申请和释放
   4).栈区：函数模块内申请，函数结束时由系统自动释放。存放局部变量、函数参数

**22. 直接调用_objc_msgForward函数将会发生什么？**
> _objc_msgForward是 IMP 类型，用于消息转发的：当向一个对象发送一条消息，但它并没有实现的时候，_objc_msgForward会尝试做消息转发。
直接调用_objc_msgForward是非常危险的事，如果用不好会直接导致程序Crash，但是如果用得好，能做很多非常酷的事。
一旦调用_objc_msgForward，将跳过查找 IMP 的过程，直接触发“消息转发”，如果调用了_objc_msgForward，即使这个对象确实已经实现了这个方法，你也会告诉objc_msgSend：“我没有在这个对象里找到这个方法的实现”

**23. 对于Run Loop的理解**
> * RunLoop，是多线程的法宝，即一个线程一次只能执行一个任务，执行完任务后就会退出线程。主线程执行完即时任务时会继续等待接收事件而不退出。非主线程通常来说就是为了执行某一任务的，执行完毕就需要归还资源，因此默认是不运行RunLoop的；
> * 每一个线程都有其对应的RunLoop，只是默认只有主线程的RunLoop是启动的，其它子线程的RunLoop默认是不启动的，若要启动则需要手动启动；
> * 在一个单独的线程中，如果需要在处理完某个任务后不退出，继续等待接收事件，则需要启用RunLoop；
> * NSRunLoop提供了一个添加NSTimer的方法，可以指定Mode，如果要让任何情况下都回调，则需要设置Mode为Common模式；
> * 实质上，对于子线程的runloop默认是不存在的，因为苹果采用了懒加载的方式。如果我们没有手动调用[NSRunLoop currentRunLoop]的话，就不会去查询是否存在当前线程的RunLoop，也就不会去加载，更不会创建。

**24. runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）**
> 1.每一个类对象中都一个对象方法列表（对象方法缓存）
   2.类方法列表是存放在类对象中isa指针指向的元类对象中（类方法缓存）
   3.方法列表中每个方法结构体中记录着方法的名称,方法实现,以及参数类型，其实selector本质就是方法名称,通过这个方法名称就可以在方法列表中找到对应的方法实现.
   4.当我们发送一个消息给一个NSObject对象时，这条消息会在对象的类对象方法列表里查找
   5.当我们发送一个消息给一个类时，这条消息会在类的Meta Class对象的方法列表里查找

**25. runtime 中，SEL 和 IMP 的区别**
> 方法名 SEL – 表示该方法的名称；
   IMP – 指向该方法的具体实现的函数指针，说白了IMP就是实现方法。

**26.block底层实现**
> block本质是指向一个结构体的一个指针
运行时机制 比较高级的特性 纯C语言
平时写的OC代码 转换成C语言运行时的代码
指令:clang -rewrite-objc  main.m(可以打印验证)
默认情况下,任何block都是在栈里面的,随时可能被回收
只要对其做一次copy操作 block的内存就会放在堆里面 不会释放
只有copy才能产生一个新的内存地址 所有地址会发生改变

**27. TCP协议三次握手**
> TCP协议采用了三次握手策略。用TCP协议把数据包送出去后，TCP不会对传送后的情况置之不理，它一定会向对方确认是否成功送达。握手过程中使用了TCP的标志——SYN(synchronize)和ACK(acknowledgement)。发送端首先发送一个带SYN标志的数据包给对方。接收端收到后，回传一个带有SYN/ACK标志的数据包以示传达确认信息。最后，发送端再回传一个带ACK标志的数据包，代表“握手”结束。
![TCP协议三次握手示意图](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.02.25.01.png)

**28. @property 的本质是什么？**
> @property = ivar + getter + setter;
“属性” (property)有两大概念：ivar（实例变量）、getter+setter（存取方法）

**29. KVC的底层实现？**
> 当一个对象调用setValue方法时，方法内部会做以下操作：
1). 检查是否存在相应的key的set方法，如果存在，就调用set方法。
2). 如果set方法不存在，就会查找与key相同名称并且带下划线的成员变量，如果有，则直接给成员变量属性赋值。
3). 如果没有找到_key，就会查找相同名称的属性key，如果有就直接赋值。
4). 如果还没有找到，则调用valueForUndefinedKey:和setValue:forUndefinedKey:方法。
这些方法的默认实现都是抛出异常，我们可以根据需要重写它们。

**30. ViewController生命周期**
> 按照执行顺序排列：
1). initWithCoder：通过nib文件初始化时触发。
2). awakeFromNib：nib文件被加载的时候，会发生一个awakeFromNib的消息到nib文件中的每个对象。      
3). loadView：开始加载视图控制器自带的view。
4). viewDidLoad：视图控制器的view被加载完成。  
5). viewWillAppear：视图控制器的view将要显示在window上。
6). updateViewConstraints：视图控制器的view开始更新AutoLayout约束。
7). viewWillLayoutSubviews：视图控制器的view将要更新内容视图的位置。
8). viewDidLayoutSubviews：视图控制器的view已经更新视图的位置。
9). viewDidAppear：视图控制器的view已经展示到window上。 
10). viewWillDisappear：视图控制器的view将要从window上消失。
11). viewDidDisappear：视图控制器的view已经从window上消失。

**31. 如何用GCD同步若干个异步调用？（如根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图）**
```
// 使用Dispatch Group追加block到Global Group Queue,这些block如果全部执行完毕，就会执行Main Dispatch Queue中的结束处理的block。
// 创建队列组
dispatch_group_t group = dispatch_group_create();
// 获取全局并发队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_async(group, queue, ^{ /*加载图片1 */ });
dispatch_group_async(group, queue, ^{ /*加载图片2 */ });
dispatch_group_async(group, queue, ^{ /*加载图片3 */ }); 
// 当并发队列组中的任务执行完毕后才会执行这里的代码
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        // 合并图片
});
```
**32. dispatch_barrier_async（栅栏函数）的作用是什么？**
```
函数定义：dispatch_barrier_async(dispatch_queue_t queue, dispatch_block_t block);
作用：
    1.在它前面的任务执行结束后它才执行，它后面的任务要等它执行完成后才会开始执行。
    2.避免数据竞争

// 1.创建并发队列
dispatch_queue_t queue = dispatch_queue_create("myQueue", DISPATCH_QUEUE_CONCURRENT);
// 2.向队列中添加任务
dispatch_async(queue, ^{  // 1.2是并行的
    NSLog(@"任务1, %@",[NSThread currentThread]);
});
dispatch_async(queue, ^{
    NSLog(@"任务2, %@",[NSThread currentThread]);
});

dispatch_barrier_async(queue, ^{
    NSLog(@"任务 barrier, %@", [NSThread currentThread]);
});

dispatch_async(queue, ^{   // 这两个是同时执行的
    NSLog(@"任务3, %@",[NSThread currentThread]);
});
dispatch_async(queue, ^{
    NSLog(@"任务4, %@",[NSThread currentThread]);
});

// 输出结果: 任务1 任务2 ——》 任务 barrier ——》任务3 任务4 
// 其中的任务1与任务2，任务3与任务4 由于是并行处理先后顺序不定。
```
