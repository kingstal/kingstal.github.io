操作队列（operation queue）是由 GCD 提供的一个队列模型的 Cocoa 抽象。**GCD 提供了更加底层的控制**，而**操作队列则在 GCD 之上实现了一些方便的功能**，这些功能对于 app 的开发者来说通常是**最好最安全的选择**。

它控制着并行操作的执行，扮演着优先级队列的角色，让它管理的高优先级操作(`NSOperation.queuePriority`)能优先于低优先级的操作运行的情况下，使它管理的操作能基本遵循先进先出的原则执行。

**NSOperationQueue** 有两种不同类型的队列：**主队列**和**自定义队列**。主队列运行在主线程之上，而自定义队列在后台执行。在两种类型中，这些队列所处理的任务都使用 **NSOperation** 的子类来表述。


`NSOperation`
表示了一个独立的计算单元。作为一个抽象类，它给了它的子类一个十分有用而且线程安全的方式来建立状态、优先级、依赖性和取消等的模型。框架还提供`NSBlockOperation`和`NSInvocationOperation`，是继承自NSOperation的实体类。

- 使用：

1. 使用提供的`NSBlockOperation`和`NSInvocationOperation`创建任务
2. 将任务添加到队列中
3. 任务间可以添加依赖，是跨队列的（`[intermediateOperation addDependency:operation1];`）、队列可以设置任务都并发数

----------------------------
可以通过重写 `main` 或者 `start` 方法 来定义自己的 `operations` 。前一种方法非常简单，开发者不需要管理一些状态属性（例如 `isExecuting` 和 `isFinished`），当 `main` 方法返回的时候，这个 operation 就结束了。这种方式使用起来非常简单，但是灵活性相对重写 `start` 来说要少一些。

    {% highlight objective-c %}
    // 重写 NSOperation 的 main 方法
    @implementation YourOperation
    - (void)main
    {
        // 进行处理 ...
    }
    @end
    {% endhighlight %}


如果你希望拥有更多的控制权，以及在一个操作中可以执行异步任务，那么就重写 `start` 方法：


    {% highlight objective-c %}
    // 重写 NSOperation 的 start 方法
    @implementation YourOperation
    - (void)start
    {
        self.isExecuting = YES;
        self.isFinished = NO;
        // 开始处理，在结束时应该调用 finished ...
    }

    - (void)finished
    {
        self.isExecuting = NO;
        self.isFinished = YES;
    }
    @end
    {% endhighlight %}

注意：这种情况下，你必须手动管理操作的状态。 为了让操作队列能够捕获到操作的改变，需要将状态的属性以配合 KVO 的方式进行实现。如果你不使用它们默认的 setter 来进行设置的话，你就需要在合适的时候发送合适的 KVO 消息。


**状态**
`NSOperation`包含了一个十分优雅的状态机来描述每一个操作的执行。
> isReady → isExecuting → isFinished

状态直接由上面那些keypath的KVO通知决定。

**取消**
当`NSOperation`的`-cancel`状态调用的时候会通过KVO通知`isCancelled`的keypath来修改`isCancelled`属性的返回值，这个时候`isCancelled`和`isFinished`的值将是`YES`，而`isExecuting`的值则为`NO`。

为了能使用操作队列所提供的取消功能，你需要在长时间操作中时不时地检查 `isCancelled` 属性：

    {% highlight objective-c %}
    // 检查 isCancelled
    - (void)main
    {
        while (notDone && !self.isCancelled) {
            // 进行处理
        }
    }
    {% endhighlight %}

**优先级**
不可能所有的`NSOperation`都是一样重要，通过以下的顺序设置`queuePriority`属性可以加快或者推迟操作的执行：

- `NSOperationQueuePriorityVeryHigh`
- `NSOperationQueuePriorityHigh`
- `NSOperationQueuePriorityNormal`
- `NSOperationQueuePriorityLow`
- `NSOperationQueuePriorityVeryLow`




当你定义好 operation 类之后，就可以很容易的将一个 operation 添加到队列中：

    {% highlight %}
    // 添加 operation 到队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    YourOperation *operation = [[YourOperation alloc] init];
    [queue  addOperation:operation];
    {% endhighlight %}

另外，你也可以将 block 添加到操作队列中。这有时候会非常的方便，比如你希望在主队列中调度一个一次性任务：

    {% highlight objective-c %}
    // 添加 block 到队列
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        // 代码...
    }];
    {% endhighlight %}

虽然通过这种的方式在队列中添加操作会非常方便，但是定义你自己的 `NSOperation` 子类会在调试时很有帮助。如果你重写 operation 的 `description` 方法，就可以很容易的标示出在某个队列中当前被调度的所有操作 。

除了提供基本的调度操作或 block 外，操作队列还提供了在 GCD 中不太容易处理好的特性的功能。例如，你可以通过 `maxConcurrentOperationCount` 属性来控制一个特定队列中可以有多少个操作参与并发执行。将其设置为 1 的话，你将得到一个串行队列，这在以隔离为目的的时候会很有用。


**依赖**
根据应用的复杂度不同，将大任务再分成一系列子任务一般都是很有意义的，而且能通过NSOperation的依赖性实现。
就是根据队列中 `operation` 的**优先级对其进行排序**，这不同于 GCD 的队列优先级，它只影响当前队列中所有被调度的 operation 的执行先后。如果你需要进一步在除了 5 个标准的优先级以外对 operation 的执行顺序进行控制的话，还可以在 operation 之间指定依赖关系，如下：

    {% highlight objective-c %}
    // operation 之间指定依赖关系
    [intermediateOperation addDependency:operation1];
    [intermediateOperation addDependency:operation2];
    [finishedOperation addDependency:intermediateOperation];
    {% endhighlight %}

这些简单的代码可以确保 `operation1` 和 `operation2` 在 `intermediateOperation` 之前执行，当然，也会在 `finishOperation` 之前被执行。对于需要明确的执行顺序时，操作依赖是非常强大的一个机制。它可以让你创建一些操作组，并确保这些操作组在依赖它们的操作被执行之前执行，或者在并发队列中以串行的方式执行操作。


**completionBlock**
每当一个`NSOperation`执行完毕，它就会调用它的`completionBlock`属性一次，这提供了一个非常好的方式让你能在视图控制器(View Controller)里或者模型(Model)里加入自己更多自己的代码逻辑。


从本质上来看，操作队列的性能比 GCD 要低那么一点，不过，大多数情况下这点负面影响可以忽略不计，**操作队列是并发编程的首选工具**。


[http://nshipster.com/nsoperation/#stq=&stp=0](http://nshipster.com/nsoperation/#stq=&stp=0)
