# 《ReactiveCocoa and MVVM, an Introduction》阅读笔记

>**不需要仅使用一个 view-model 代表屏幕上展示的所有东西！** 你可以使用 child view-model 表示更小的、潜在的更具封装性的部分：如果某一小块视图(比如 cell)在你的app中可以复用，或者它表示多个 data-model 对象，这么做将会十分有益。



>严格来讲，你应该在 app delegate 中为顶级视图控制器创建一个 view-model。当展示一个新的 view controller 或者一个很小的视图(这个小的视图使用 view-model 表示)时，要让当前的这个 view-model 为你创建需要的 child view-model 。



> 假如我们想要在用户点击应用顶部的头像时，添加一个资料视图控制器，我们可以为当前主 view-model 添加一个方法：
>
> ```objective-c
> - (MYTwitterUserProfileViewModel *) viewModelForCurrentUser;
> ```

> 在我们的主控制器中，可以像这样使用它：
>
> ```objective-c
> - (IBAction) didTapPrimaryUserAvatar
> {
>     MYTwitterUserProfileViewModel *userProfileViewModel = [self.viewModel viewModelForCurrentUser];
>     
>     MYTwitterUserProfileViewController *profileViewController = 
>         [[MYTwitterUserProfileViewController alloc] initWithViewModel: userProfileViewModel];
>     [self.navigationController pushViewController: profileViewController animated:YES];
> }
> ```



> 不管应用程序中发生了什么，相同的输入，就会得到相同的输出。这就是 函数式



> **我们要做的第一步就是在你的view-model的头文件中不要包含UIKit.h**（这是一个很好的原则，但也有一些灰色区域：比如，你可能会将UIImage看作数据，而不是视图（我喜欢这样）。在这种情况下，你需要UIKit.h来获得UIImage类



> 有时你无法在初始化时将 view-model 传入, 比如在 storyboard segue 或 cell dequeuing 的情况下. 这时你应该在该 view (controller) 中暴露一个公有可写的 view-model 属性.

```
MYTwitterUserCell *cell =
    [self.tableView dequeueReusableCellWithIdentifier:@"MYTwitterUserCell" forIndexPath:indexPath];
// grab the cell view-model from the vc view-model and assign it
cell.viewModel = self.viewModel.tweets[indexPath.row];
```



> 现在我们开始看 RAC 为我们提供的转换数据流的值的方法。我们会用到 `RACSignal` 类提供的 `map` 实例方法。

```
- (void) viewDidLoad {
    //...
    RACSignal *usernameIsValidSignal = RACObserve(self.viewModel, isUsernameValid);
    RAC(self.goButton, enabled) = usernameIsValidSignal;
    RAC(self.goButton, alpha) = [usernameIsValidSignal
        map:^id(NSNumber *usernameIsValid) {
            return usernameIsValid.boolValue ? @1.0 : @0.5;
        }];
}
```



> 每当一个新值通过该信号链发送时，它实际上是每一个订阅者都会发送一次，比如新增了一个订阅者去监听一个信号，那么信号会立即向订阅者发送信息，注意是所有订阅者！而不仅仅是你刚才新增的那个。信号发送出的信息(值)不会存储在任何地方(除了RAC的内部实现部分)



> 我们需要同时追踪 `tweets` 和 `allTweetsLoaded` 这两个属性，目的是为了防止其中一个发生变化而另一个没有变(有一个属性变化，就需要更新 table view)。问题是，有时这两个属性碰巧会在同一时间点发生变化，这意味着，合并产生的大信号中的两个小信号都将发送一个值，那么`reloadData` 将在同一个 run loop 中被连续调用两次。UIKit 不喜欢这样。`bufferWithTime:` 捕获任何在给定时间内发送过来的下一个值，当这段时间过去以后，再将这些值一并发送给订阅者。通过传入参数 0 ，`bufferWithTime:`方法将捕获在一个 run-loop 时间内由“大信号”发出的所有值，然后再将这些值一起发出去。别担心，就把它当做需要将这些值必须在主线程上传递。现在我们能够确保 `reloadData` 方法在每个run-loop 内只执行一次。

```
@weakify(self);
[[[RACSignal merge:@[RACObserve(self.viewModel, tweets),
                     RACObserve(self.viewModel, allTweetsLoaded)]]
    bufferWithTime:0 onScheduler:[RACScheduler mainThreadScheduler]]
    subscribeNext:^(id value) {
        @strongify(self);
        [self.tableView reloadData];
    }];
```