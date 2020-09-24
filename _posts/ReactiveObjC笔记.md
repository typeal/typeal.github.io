| layout | title            | subtitle     | date       | author | header-img               | catalog | tags |
| ------ | ---------------- | ------------ | ---------- | ------ | ------------------------ | ------- | ---- |
| post   | ReactiveObjC笔记 | just a test! | 2020-03-08 | Ash    | img/post-bg-ios9-web.jpg | true    | Test |



# ReactiveObjC笔记

**只有支持KVO的property才能被RACObserve**



> **多信号组合使用**
>
> ``` 
>     RAC(self.logInButton, enabled) = [RACSignal combineLatest:@[self.usernameTextField.rac_textSignal,
> 		self.passwordTextField.rac_textSignal,
> 		RACObserve(LoginManager.sharedManager, loggingIn),
> 		RACObserve(self, loggedIn)] reduce:^(NSString *username, 
> 																				 NSString *password, 
> 																				 NSNumber *loggingIn, 
> 																				 NSNumber *loggedIn) {
>     return @(username.length > 0 && password.length > 0 && 
>     				!loggingIn.boolValue && !loggedIn.boolValue);
>     }];
> ```



> ```
> return [RACDisposable disposableWithBlock:^{
>          // block调用时刻：当信号发送完成或者发送错误，就会自动执行这个block,取消订阅信号。
>          // 执行完Block后，当前信号就不再被订阅了。
>          NSLog(@"信号被销毁");
>      }];
> ```



> flattenMap作用:把源信号的内容映射成一个新的信号，信号可以是任意类型。（可以用于解包信号）
>
> flattenMap使用步骤:
> 传入一个block，block类型是返回值RACStream，参数value
> 参数value就是源信号的内容，拿到源信号的内容做处理
> 包装成RACReturnSignal信号，返回出去。



> 拼接信号：[signalA concat:signalB] 第一个信号必须发送完成，第二个信号才会被激活 

> then:用于连接两个信号，当第一个信号完成后,过滤掉第一个信号的值，执行then返回的信号

> merge 合并两个信号，任何一个信号发送数据都能被监听到



> RACCommand需要被强引用，信号是延迟发送的，可能发送之前command就被释放掉了
>
> 如果数据传输完需要发送completion，否则命令永远处于执行状态
>
> RACCommand不支持并发操作，需要上一个操作完成后才能执行下一个操作



> replay和replayLast使信号变成热信号，且会提供所有值(`-replay`) 或者最新的值(`-replayLast`) 给订阅者。 replayLazily返回一个冷的信号，会提供所有的值给订阅者（这三个都会将值缓存起来，不会重新生成或请求数据）



rac_prepareForReuseSignal（Cell复用信号）



**热信号是主动的，即使你没有订阅事件，它仍然会时刻推送，而冷信号是被动的，只有当你订阅的时候，它才会发送消息 **

**热信号可以有多个订阅者，是一对多，信号可以与订阅者共享信息，而冷信号只能一对一，当有不同的订阅者，消息会从新完整发送**

**任何的信号转换即是对原有的信号进行订阅从而产生新的信号**



**RAC中Pull-driver和Push-driver的区别？**

> Pull-driver是指的是任何时刻，我们如果需要数据了，都可以从pull-driver里面拿走数据，因为数据先存储了。整个取数据的时间控制在调用者手上。典型的例子就是for-in循环，这就是一个pull-driver的操作。不管你循环几次，每次循环如何操作，数组或者字典里面的数据都一直存在在那里，“躺”在那里。
> Push-driver是相反的，在任何时刻，当有数据或者事件产生，都会push给你，如果你此时没有处理，该事件或者数据就丢失了。整个取数据的时间并不控制在调用者的手里。
>
> Pull-driver可以类比看书，知识和文字不管你看不看，一直都在书里。
> Push-driver可以类比看电视，节目不管你看不看，都一直播放，你错过了就是错过了。
>
> 在RAC里面，Sequence就是一个pull-driver，Subject就是一个push-driver。



1.RACSubject`及其子类是**热信号**。

2.RACSignal`排除`RACSubject`类以外的是**冷信号**。

RACSubject必须先订阅了才能发送数据，否则订阅者接收不到数据。而RACReplaySubject可以先发送数据也可以先订阅

> 一种将冷信号转换为热信号的方法就是，将冷信号订阅，订阅到的每一个时间通过`RACSbuject`发送出去，其他订阅者只订阅这个`RACSubject`



处理错误：catchTo