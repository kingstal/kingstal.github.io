### 队列类型
1. 主队列（**main queue**）：系统提供了一个叫做 `主队列（main queue）`的特殊队列，它和**串行队列**一样，在同一时间只能执行一个任务。然而，能保证所有的任务都在主线程上执行，只有主线程是唯一一个被允许去刷新UI的线程。这个队列是**用于给UIView发送消息或推送通知**。
2. 并发队列：系统提供了几个并发线程。比如说：`全局调度队列（Global Dispatch Queues）`。目前全局队列有四种不同的优先级：后台`（background）`、低`（low）`、默认`（default）`和高`（high）`。你应该要明白，Apple的API也会使用这些队列，所以你**添加的任何任务都不会是这些队列中唯一的任务**。
3. 自己创建的串行或并行队列

> - `dispatch_queue_t dispatch_get_main_queue ( void );`
- `dispatch_queue_t dispatch_get_global_queue ( long identifier, unsigned long flags );`



### 后台任务

> - `void dispatch_async ( dispatch_queue_t queue, dispatch_block_t block );`

提交一个异步执行的任务`block`到队列`queue`，然后立即返回。任务会在之后由 GCD 决定执行。当需要在后台执行一个基于网络或 CPU 紧张的任务时就使用 **dispatch_async** ，这样就不会阻塞当前线程。

下面是一个关于在 `dispatch_async` 上如何以及何时使用不同的队列类型的快速指导：


1. **自定义串行队列**：当你想串行执行后台任务并追踪它时就是一个好选择。这消除了资源争用，因为你知道一次只有一个任务在执行。注意若你需要来自某个方法的数据，你必须内联另一个 Block 来找回它或考虑使用 dispatch_sync。
2. **主队列（串行）**：这是在一个并发队列上完成任务后**更新 UI** 的共同选择。要这样做，你将在一个 Block 内部编写另一个 Block 。以及，如果你在主队列调用 dispatch_async 到主队列，你能确保这个新任务将在当前方法完成后的某个时间执行。
3. **并发队列**：这是在后台执行非 UI 工作的共同选择。


### 延后任务

> - `typedef uint64_t dispatch_time_t;`时间的抽象表示
> - `dispatch_time_t dispatch_time ( dispatch_time_t when, int64_t delta );`创建一个相对于`when`的`dispatch_time_t`
> - `void dispatch_after ( dispatch_time_t when, dispatch_queue_t queue, dispatch_block_t block );`直到给定的时间，添加异步任务`block`到一个给定的队列`queue`。
`dispatch_after` 工作起来就像一个延迟版的 `dispatch_async` 。依然不能控制实际的执行时间，且一旦 `dispatch_after` 返回也就不能再取消它。

如何在`dispatch_after`上使用不同的队列类型：
1. **自定义串行队列**：在一个自定义串行队列上使用 dispatch_after 要小心。你最好坚持使用主队列。
2. **主队列（串行）**：是使用 dispatch_after 的好选择；Xcode 提供了一个不错的自动完成模版。
3. **并发队列**：在并发队列上使用 dispatch_after 也要小心；你会这样做就比较罕见。还是在主队列做这些操作吧。


### 单例
> - `typedef long dispatch_once_t;`变量必须是`global`或`static`，和 `dispatch_once`一起使用
> - `void dispatch_once ( dispatch_once_t *predicate, dispatch_block_t block );`在整个应用程序生命周期，`block`执行且只执行一次，用于创建单例


### 读者与写者

GCD 通过用 `dispatch barriers` 创建一个读者写者锁。
`Dispatch barriers` 是一组函数，在并发队列上工作时扮演一个串行式的瓶颈。使用 GCD 的障碍（barrier）API 确保提交的 Block 在那个特定时间上是指定队列上唯一被执行的条目。这就意味着所有的先于调度障碍提交到队列的条目必能在这个 Block 执行前完成。

当这个 Block 的时机到达，调度障碍执行这个 Block 并确保在那个时间里队列不会执行任何其它 Block 。一旦完成，队列就返回到它默认的实现状态。 GCD 提供了**同步**和**异步**两种障碍函数。

下面是何时会——和不会——使用障碍函数的情况：
1. 自定义串行队列：一个很坏的选择；障碍不会有任何帮助，因为不管怎样，一个串行队列一次都只执行一个操作。
2. 全局并发队列：要小心；这可能不是最好的主意，因为其它系统可能在使用队列而且你不能垄断它们只为你自己的目的。
3. **自定义并发队列**：这对于原子或临界区代码来说是极佳的选择。任何你在设置或实例化的需要线程安全的事物都是使用障碍的最佳候选。

> - `void dispatch_barrier_async ( dispatch_queue_t queue, dispatch_block_t block );`异步提交一个`barrier block`到队列，立即返回


    {% highlight objective-c %}
    // 写者
    - (void)addPhoto:(Photo *)photo,
    {
        if (photo) { // 1. 在执行下面所有的工作前检查是否有合法的相片
            dispatch_barrier_async(self.concurrentPhotoQueue, ^{ // 2. 添加写操作到你的自定义队列。当临界区在稍后执行时，这将是你队列中唯一执行的条目。
                [_photosArray addObject:photo]; // 3. 这是添加对象到数组的实际代码。由于它是一个障碍 Block ，这个 Block 永远不会同时和其它 Block 一起在 concurrentPhotoQueue 中执行。
                dispatch_async(dispatch_get_main_queue(), ^{ // 4. 发送一个通知说明完成了添加图片。这个通知将在主线程被发送因为它将会做一些 UI 工作，所以在此为了通知，你异步地调度另一个任务到主线程。
                    [self postContentAddedNotification];
                });
            });
        }
    }
    {% endhighlight %}


> - `void dispatch_sync ( dispatch_queue_t queue, dispatch_block_t block );`
`dispatch_sync()` 同步地提交工作并在返回前等待它完成。使用 `dispatch_sync` 跟踪你的调度障碍工作，或者当你需要等待操作完成后才能使用 Block 处理过的数据。如果你使用第二种情况做事，你将不时看到一个 `__block` 变量写在 `dispatch_sync` 范围之外，以便返回时在 `dispatch_sync` 使用处理过的对象。

下面是一个快速总览，关于在何时以及何处使用 `dispatch_sync` ：

1. 自定义串行队列：在这个状况下要非常小心！如果你正运行在一个队列并调用 dispatch_sync 放在同一个队列，那你就百分百地创建了一个死锁。
2. 主队列（串行）：同上面的理由一样，必须非常小心！这个状况同样有潜在的导致死锁的情况。
3. **并发队列**：这才是做同步工作的好选择，不论是通过调度障碍，或者需要等待一个任务完成才能执行进一步处理的情况。


    {% highlight objective-c %}
    // 读者
    - (NSArray *)photos
    {
        __block NSArray *array; // 1.  __block 关键字允许对象在 Block 内可变。没有它，array 在 Block 内部就只是只读的，你的代码甚至不能通过编译。
        dispatch_sync(self.concurrentPhotoQueue, ^{ // 2. 在 concurrentPhotoQueue 上同步调度来执行读操作。
            array = [NSArray arrayWithArray:_photosArray]; // 3. 将相片数组存储在 array 内并返回它。
        });
        return array;
    }
    {% endhighlight %}


最后实例化自定义队列`concurrentPhotoQueue`：

> - `typedef struct dispatch_queue_s *dispatch_queue_t;`轻量级对象，应用程序提交`block`到该对象执行
> - `dispatch_queue_t dispatch_queue_create ( const char *label, dispatch_queue_attr_t attr );`创建一个自定义的队列


    {% highlight objective-c %}
    // 实例化自定义队列
    sharedPhotoManager->_concurrentPhotoQueue = dispatch_queue_create("com.selander.GooglyPuff.photoQueue",DISPATCH_QUEUE_CONCURRENT);
    //使用 dispatch_queue_create 初始化 concurrentPhotoQueue 为一个并发队列。第一个参数是反向DNS样式命名惯例；确保它是描述性的，将有助于调试。第二个参数指定你的队列是串行还是并发。
    {% endhighlight %}


### 调度组（dispatch_group）

`Dispatch Group` 会在整个组的任务都完成时通知你。这些任务可以是同步的，也可以是异步的，即便在不同的队列也行。而且在整个组的任务都完成时，Dispatch Group 可以用同步的或者异步的方式通知你。因为要监控的任务在不同队列，那就用一个 dispatch_group_t 的实例来记下这些不同的任务。

当组中所有的事件都完成时，GCD 的 API 提供了两种通知方式。

**1.dispatch_group_wait**：它会阻塞当前线程，直到组里面所有的任务都完成或者等到某个超时发生。


    {% highlight objective-c %}
    // dispatch_group_wait
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{ // 1. 因为你在使用的是同步的 dispatch_group_wait ，它会阻塞当前线程，所以你要用 dispatch_async 将整个方法放入后台队列以避免阻塞主线程。

        __block NSError *error;
        dispatch_group_t downloadGroup = dispatch_group_create(); // 2. 创建一个新的 Dispatch Group，它的作用就像一个用于未完成任务的计数器。

        for (NSInteger i = 0; i < 3; i++) {
            NSURL *url;
            switch (i) {
                case 0:
                    url = [NSURL URLWithString:kOverlyAttachedGirlfriendURLString];
                    break;
                case 1:
                    url = [NSURL URLWithString:kSuccessKidURLString];
                    break;
                case 2:
                    url = [NSURL URLWithString:kLotsOfFacesURLString];
                    break;
                default:
                    break;
            }

            dispatch_group_enter(downloadGroup); // 3. dispatch_group_enter 手动通知 Dispatch Group 任务已经开始。你必须保证 dispatch_group_enter 和 dispatch_group_leave 成对出现，否则你可能会遇到诡异的崩溃问题。
            Photo *photo = [[Photo alloc] initwithURL:url
                                  withCompletionBlock:^(UIImage *image, NSError *_error) {
                                      if (_error) {
                                          error = _error;
                                      }
                                      dispatch_group_leave(downloadGroup); // 4. 手动通知 Group 它的工作已经完成。再次说明，你必须要确保进入 Group 的次数和离开 Group 的次数相等。
                                  }];

            [[PhotoManager sharedManager] addPhoto:photo];
        }
        dispatch_group_wait(downloadGroup, DISPATCH_TIME_FOREVER); // 5. dispatch_group_wait 会一直等待，直到任务全部完成或者超时。如果在所有任务完成前超时了，该函数会返回一个非零值。你可以对此返回值做条件判断以确定是否超出等待周期；然而，你在这里用 DISPATCH_TIME_FOREVER 让它永远等待。它的意思，勿庸置疑就是，永－远－等－待！这样很好，因为图片的创建工作总是会完成的。
        dispatch_async(dispatch_get_main_queue(), ^{ // 6. 此时此刻，你已经确保了，要么所有的图片任务都已完成，要么发生了超时。然后，你在主线程上运行 completionBlock 回调。这会将工作放到主线程上，并在稍后执行。
            if (completionBlock) { // 7. 最后，检查 completionBlock 是否为 nil，如果不是，那就运行它。
                completionBlock(error);
            }
        });
    });
    {% endhighlight %}


> - `typedef struct dispatch_group_s *dispatch_group_t;`：调度组（group of block）
> - `dispatch_group_t dispatch_group_create ( void );`：创建调度组
> - `void dispatch_group_enter ( dispatch_group_t group );`：block进入调度组
> - `void dispatch_group_leave ( dispatch_group_t group );`：block离开调度组，成对出现
> - `long dispatch_group_wait ( dispatch_group_t group, dispatch_time_t timeout );`：同步等待`group`中的`block`完成，知道 timeout，会阻塞当前线程


**2.dispatch_group_notify**：异步方式通知

    {% highlight objective-c %}
    // dispatch_group_notify
    // 1. 在新的实现里，因为你没有阻塞主线程，所以你并不需要将方法包裹在 async 调用中。
    __block NSError *error;
    dispatch_group_t downloadGroup = dispatch_group_create();

    for (NSInteger i = 0; i < 3; i++) {
        NSURL *url;
        switch (i) {
            case 0:
                url = [NSURL URLWithString:kOverlyAttachedGirlfriendURLString];
                break;
            case 1:
                url = [NSURL URLWithString:kSuccessKidURLString];
                break;
            case 2:
                url = [NSURL URLWithString:kLotsOfFacesURLString];
                break;
            default:
                break;
        }

        dispatch_group_enter(downloadGroup); // 2. 同样的 enter 方法，没做任何修改。
        Photo *photo = [[Photo alloc] initwithURL:url
                              withCompletionBlock:^(UIImage *image, NSError *_error) {
                                  if (_error) {
                                      error = _error;
                                  }
                                  dispatch_group_leave(downloadGroup); // 3. 同样的 leave 方法，也没做任何修改。
                              }];

        [[PhotoManager sharedManager] addPhoto:photo];
    }

    dispatch_group_notify(downloadGroup, dispatch_get_main_queue(), ^{ // 4. dispatch_group_notify 以异步的方式工作。当 Dispatch Group 中没有任何任务时，它就会执行其代码，那么 completionBlock 便会运行。你还指定了运行 completionBlock 的队列，此处，主队列就是你所需要的。
        if (completionBlock) {
            completionBlock(error);
        }
    });
    {% endhighlight %}


> - `void dispatch_group_notify ( dispatch_group_t group, dispatch_queue_t queue, dispatch_block_t block );`：不会阻塞当前线程
