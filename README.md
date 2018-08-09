# SummingUpKnowledge
知识点总结
RunTime:
      1. objc_msgSend 功能:在OC中，消息不跟方法实现绑定直到运行时。编译器将消息表达式 [receiver message] 转化成一个消息传递函数objc_msgSend。这个函数将接收者和在消息中提到的方法名（方法选择器）作为他的两个主要参数：objc_msgSend(receiver, selector)。消息中任何参数也交给objc_msgSend：objc_msgSend(receiver, selector, arg1, arg2, …)。
消息传递函数为动态绑定做了所有必须的事情：它首先发现方法选择器指向的程序（方法的实现）。因为相同的方法可以被不同的类分别实现。这个准确的程序依赖于接收者的类。然后调用程序，通过接收对象（指针指向他的数据）为方法传递指定的参数。
最后，当他返回值的时候它传递程序的返回值。
