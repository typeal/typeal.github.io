# 响应式编程（Reactive Programming  RP）

### 定义

**响应式编程（reactive programming）是一种基于数据流（data stream）和变化传递（propagation of change）的声明式（declarative）的编程范式**



### RP是使用异步数据流进行编程

> 用包括Click或网络请求事件在内的任何东西创建Data stream（数据流），任何东西都可以是一个Stream：变量、用户输入、属性、Cache、数据结构

> Stream就是一个 **按时间排序的Events(Ongoing events ordered in time)序列** ，它可以emit（发出）三种不同的Events：(某种类型的)Value、Error或者一个"Completed" Signal。考虑一下"Completed"发生的时机，例如，当包含这个Button(指上面Clicks on a button"例子中的Button)的Window或者View被关闭时。



### 概念

**一种基于事件的模型，将上一个任务结果的反馈看作一个事件，这个事件的到来会触发下一个任务**

> 这也就是 Reactive 的内涵。我们把处理和发出事件的主体称为 Reactor，它可以接收事件并处理，也可以在处理完事件后，发出下一个事件给其他 Reactor。两个 Reactors 之间没有必然的耦合，他们之间通过消息管道（channel）来传递消息。Reactor 可以预先定义一些事件处理函数，根据接收到的事件不同类型来进行不同的处理。如果我们的系统复杂，我们还可以专门定义不同功能类别的 Reactors，分别处理不同类型的事件。在每个 Reactor 中，我们还可以对事件再进行细分处理

实现Reactive模型最核心的是线程与消息管道

线程用于侦听事件，消息管道用于 Reactor 之间通信不同的消息。与他们相关的是事件管理器用于注册、注销事件，而消息分配器则会根据消息类型分发

错误（未定义）事件则可以单独放在一个专门处理 Error/Exception 的 Reactor 中