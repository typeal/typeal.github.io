- 将数组转化成一个序列类型的流。
- 对流进行映射得到一个新的流。
- 将新的流转为数组。

序列，默认情况下是延迟加载的(也称：懒加载或被动加载)，是`pull-driven`的，在他们被生成的时候就会提供确切的值，而数组方法会强制给序列的每一个成员赋值。



> Push-driven : 在创建信号的时候，信号不会被立即赋值，之后才会被赋值(举个栗子：网络请求回来的结果或者是任意的用户输入的结果)

> Pull-driven : 在创建信号的同时序列中的值就会被确定下来，我们可以从流中一个个地查询值。



Next Error Completion

> 一个事情响应中，一个`signal`(信号)发送了一个`Error value`或者一个`Completion Value`后，就不会再发送任何其他的`value`. 错误或者成功将只会发送其中一个，绝不会有两个同时发送的情况！



> 指令，RACCommand类的代表，创建并订阅动作的信号响应，可以很容易地实现一些用户与应用交互时的边界效果。

> 指令(行为触发的)通常是UI驱动的，比如按键的点击。指令也可以通过信号自动禁用，这种禁用状态呈现在UI上就是禁用与该指令相关联的任何操作。



> 每增加一个订阅都会发送一次信号



> RACReplaySubject可以确保他背后的Subject只会被订阅一次,RACReplaySubject将会缓存这个订阅的值，并将其转发给新的订阅者们-



> 创建NSDateFormatter代价昂贵



>  举一个具体的例子：假设我们的模型包含一个“日期”(用`dateAdded`表示)，我们想要监控这个“日期”的变化，来更新我们视图模型(viewModel)的属性`dateAdded`.模型(model)的属性是一个`NSDate`的实例，但视图模型(viewModel)中对应的属性是从它转换过来的`NSString`。这种绑定看起来跟下面的代码类似(在viewModel的初始化方法中进行)：
>
> ```
> RAC(self, dateAdded) = [RACObserve(self.model,dateAdded) map:^(NSDate *date) {
> 	return [[ViewModel dateFormatter] stringFromDate:date];
> }];
> ```



