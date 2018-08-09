# SummingUpKnowledge

知识点总结


### objc_msgSend 功能

首先，编译器将代码[obj makeText];转化为objc_msgSend(obj, @selector (makeText));，在objc_msgSend函数中。首先通过obj的isa指针找到obj对应的class。在Class中先去cache中 通过SEL查找对应函数method（猜测cache中method列表是以SEL为key通过hash表来存储的，这样能提高函数查找速度），若 cache中未找到。再去methodList中查找，若methodlist中未找到，则取superClass中查找。若能找到，则将method加 入到cache中，以方便下次查找，并通过method中的函数指针跳转到对应的函数中去执行。

下面讲讲消息传递用到的一些概念：

* 类对象(objc_class)  Objective-C类是由Class类型来表示的，它实际上是一个指向objc_class结构体的指针。struct objc_class {
Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
Class _Nullable super_class                              OBJC2_UNAVAILABLE;
const char * _Nonnull name                               OBJC2_UNAVAILABLE;
long version                                             OBJC2_UNAVAILABLE;
long info                                                OBJC2_UNAVAILABLE;
long instance_size                                       OBJC2_UNAVAILABLE;
struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;

* 实例(objc_object) 就是从isa指针指向的结构体创建，类对象的isa指针指向的我们称之为元类(metaclass)，struct objc_object {
Class isa  OBJC_ISA_AVAILABILITY;
};
* 元类(Meta Class)
* Method(objc_method)
* SEL(objc_selector)
* IMP
* 类缓存(objc_cache)
* Category(objc_category)  Category是表示一个指向分类的结构体的指针，其定义如下：从上面的category_t的结构体中可以看出，分类中可以添加实例方法，类方法，甚至可以实现协议，添加属性，不可以添加成员变量。

* IMP 在iOS的Runtime中，Method通过selector和IMP两个属性，实现了快速查询方法及实现，相对提高了性能，又保持了灵活性。就是指向最终实现程序的内存地址的指针。

*  在 Objective-C 的运行时中，每个类有两个方法都会自动调用。+load 是在一个类被初始装载时调用，+initialize 是在应用第一次调用该类的类方法或实例方法前调用的。两个方法都是可选的，并且只有在方法被实现的情况下才会被调用。





### 运行时的作用

* 能获得某个类的所有成员变量
* 能获得某个类的所有属性
* 能获得某个类的所有方法
* 交换方法实现
* 能动态添加一个成员变量
* 能动态添加一个属性
* 字典转模型

### KVO实现
* 全称是Key-value observing，翻译成键值观察。提供了一种当其它对象属性被修改的时候能通知当前对象的机制。再MVC大行其道的Cocoa中，KVO机制很适合实现model和controller类之间的通讯。
* KVO的实现依赖于 Objective-C 强大的 Runtime，当观察某对象 A 时，KVO 机制动态创建一个对象A当前类的子类，并为这个新的子类重写了被观察属性 keyPath 的 setter 方法。setter 方法随后负责通知观察对象属性的改变状况。

* Apple 使用了 isa-swizzling 来实现 KVO 。当观察对象A时，KVO机制动态创建一个新的名为：NSKVONotifying_A的新类，该类继承自对象A的本类，且 KVO 为 NSKVONotifying_A 重写观察属性的 setter 方法，setter 方法会负责在调用原 setter 方法之前和之后，通知所有观察对象属性值的更改情况。


### RunLoop 的概念
* 一般来讲，一个线程一次只能执行一个任务，执行完成后线程就会退出。如果我们需要一个机制，让线程能随时处理事件但并不退出，通常的代码逻辑是这样的：
function loop() {
initialize();
do {
var message = get_next_message();
process_message(message);
} while (message != quit);
}
这种模型通常被称作 Event Loop。

* 线程和 RunLoop 之间是一一对应的，其关系是保存在一个全局的 Dictionary 里。线程刚创建时并没有 RunLoop，如果你不主动获取，那它一直都不会有。RunLoop 的创建是发生在第一次获取时，RunLoop 的销毁是发生在线程结束时。你只能在一个线程的内部获取其 RunLoop（主线程除外）。

* 一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer。每次调用 RunLoop 的主函数时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode。如果需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入。这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。

### 苹果用 RunLoop 实现的功能
* AutoreleasePool
* 事件响应
* 手势识别
* 界面更新
* 定时器
* PerformSelecter
* 关于GCD

### Autorelease Pool

 * Autorelase Pool 提供了一种可以允许你向一个对象延迟发送 release 消息的机制。当你想放弃一个对象的所有权，同时又不希望这个对象立即被释放掉（例如在一个方法中返回一个对象时），Autorelease Pool 的作用就显现出来了。
    所谓的延迟发送 release 消息指的是，当我们把一个对象标记为 autorelease 时: NSString* str = [[[NSString alloc] initWithString:@"hello"] autorelease];
    这个对象的 retainCount 会+1，但是并不会发生 release。当这段语句所处的 autoreleasepool 进行 drain 操作时，所有标记了 autorelease 的对象的 retainCount 会被 -1。即 release 消息的发送被延迟到 pool 释放的时候了。
    Autorelease Pool 的用处
    在 ARC 下，我们并不需要手动调用 autorelease 有关的方法，甚至可以完全不知道 autorelease 的存在，就可以正确管理好内存。因为 Cocoa Touch 的 Runloop 中，每个 runloop circle 中系统都自动加入了 Autorelease Pool 的创建和释放。
    
    autorelease
    
    * 给对象发送一条autorelease消息，会将对象放到一个自动释放池中。当自动释放池被销毁时，会对池子里面的所有对象做一次release操作 ，会返回对象本身，调用完autorelease方法后，对象的计数器不变
    
    使用自动释放池需要注意：
    1）自动释放池实质上只是在释放的时候給池中所有对象对象发送release消息，不保证对象一定会销毁，如果自动释放池向对象发送release消息后对象的引用计数仍大于1，对象就无法销毁。
    
    2）自动释放池中的对象会集中同一时间释放，如果操作需要生成的对象较多占用内存空间大，可以使用多个释放池来进行优化。比如在一个循环中需要创建大量的临时变量，可以创建内部的池子来降低内存占用峰值。
    
    3）autorelease不会改变对象的引用计数
    

### 引用计数
* 在ObjC中，对象什么时候会被释放（或者对象占用的内存什么时候会被回收利用）？
答案是：当对象没有被任何变量引用（也可以说是没有指针指向该对象）的时候，就会被释放。
* 那怎么知道对象已经没有被引用了呢？
ObjC采用引用计数（reference counting）的技术来进行管理：
　　　　1）每个对象都有一个关联的整数，称为引用计数器
　　　　2）当代码需要使用该对象时，则将对象的引用计数加1
　　　　3）当代码结束使用该对象时，则将对象的引用计数减1
　　　　4）当引用计数的值变为0时，表示对象没有被任何代码使用，此时对象将被释放。
　　　　
　　　与之对应的消息发送方法如下：
　　　　1）当对象被创建（通过alloc、new或copy等方法）时，其引用计数初始值为1
　　　　2）給对象发送retain消息，其引用计数加1
　　　　3）給对象发送release消息，其引用计数减1
　　　　4）当对象引用计数归0时，ObjC給对象发送dealloc消息销毁对象

### weak
   *  weak基本用法
      weak是弱引用，用weak描述修饰或者所引用对象的计数器不会加一，并且会在引用的对象被释放的时候自动被设置为nil，大大避免了野指针访问坏内存引起崩溃的情况，另外weak还可以用于解决循环引用。
  * weak原理概括
    weak表其实是一个hash（哈希）表，Key是所指对象的地址，Value是weak指针的地址数组。weak的底层实现的原理是什么？
    Runtime维护了一个weak表，用于存储指向某个对象的所有weak指针。weak表其实是一个hash表，Key是所指对象的地址，value是weak指针的地址（这个地址的值是所指对象指针的地址）数组。
    为什么value是数组？因为一个对象可能被多个弱引用指针指向
  
  当weak引用指向的对象被释放时，又是如何去处理weak指针的呢？当释放对象时，其基本流程如下：

  1、调用objc_release
  2、因为对象的引用计数为0，所以执行dealloc
  3、在dealloc中，调用了_objc_rootDealloc函数
  4、在_objc_rootDealloc中，调用了object_dispose函数
  5、调用objc_destructInstance
  6、最后调用objc_clear_deallocating,详细过程如下：
  a. 从weak表中获取废弃对象的地址为键值的记录
  b. 将包含在记录中的所有附有 weak修饰符变量的地址，赋值为   nil
  c. 将weak表中该记录删除
  d. 从引用计数表中删除废弃对象的地址为键值的记录
    
### HTTP的特性
  
HTTP构建于TCP/IP协议之上，默认端口号是80
HTTP是无连接无状态的

### TCP的特性
 
 * 三次握手与四次挥手
 所谓三次握手(Three-way Handshake)，是指建立一个 TCP 连接时，需要客户端和服务器总共发送3个包。
 三次握手的目的是连接服务器指定端口，建立 TCP 连接，并同步连接双方的序列号和确认号，交换 TCP 窗口大小信息。在 socket 编程中，客户端执行 connect() 时。将触发三次握手。
 

### Socket 基本概念
Socket 是对 TCP/IP 协议族的一种封装，是应用层与TCP/IP协议族通信的中间软件抽象层。从设计模式的角度看来，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

Socket 还可以认为是一种网络间不同计算机上的进程通信的一种方法，利用三元组（ip地址，协议，端口）就可以唯一标识网络中的进程，网络中的进程通信可以利用这个标志与其它进程进行交互。



　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　




