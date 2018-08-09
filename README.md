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




