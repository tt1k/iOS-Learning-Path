### 第五章: 内存管理

#### No.29: 理解引用计数

* ARC下

  * 当OC对象的引用计数值为0时，OS会在下一次event loop中对其执行release操作
  * 需要注意循环引用问题，避免发生内存泄漏

* MRC下

  * 当我们手动release对象后应该对其置nil，防止野指针异常

    ```objective-c
    NSNumber *number = [[NSNumber alloc] initWithInt:1337];
    [array addObject:number];
    [number release];
    number = nil;
    ```

  * 编写setter时应该先对新值retain，再对旧值release，防止当新旧指针为同一个对象时发生异常

    ```objective-c
    - (void)setFoo:(id)foo {
        [foo retain];
        [_foo release];
        _foo = foo;
    }
    ```



#### No.30: 用ARC简化引用计数

* ARC实质上是根据我们指定的内存管理语义，在对象生命周期中合适的地方插入retain和release等方法

  ARC通过底层C函数直接调用retain/release等方法，以加快执行效率，因此override这些方法无效

* ARC只负责OC对象的内存管理，我们使用CoreFoundation对象时仍然需要手动CFRelease/CFRetain

* 关于dealloc，我们不应该在dealloc里调用超类dealloc，因为ARC借助Objective-C++的cleanup routine调用所有对象的destructor，来生成.cxx_destruct方法清理内存

  但CoreFoundation对象和由malloc()分配在堆中的内存仍然需要我们释放

  ```objective-c
  - (void)dealloc {
      CFRelease(_coreFoundationObject);
      free(_heapAllocatedMemoryBlob);
  }
  ```



#### No.31: 在dealloc中仅释放引用和解除监听

* dealloc中要做的事情应该尽量少，例如释放指向其他对象的引用、解除KVO和NSNotificationCenter

  不应该在dealloc中执行异步操作，因为对象正在回收中，可能导致异常

* 若对象持有文件描述符、套接字和大块内存等开销较大或系统稀缺的资源，应该专门编写close方法，并约定调用者用完资源后必须手动close，不应该指望在dealloc中才释放



#### No.32: 编写exception-safe code时留意内存管理问题

* 纯C没有异常，C++和OC都支持异常且互通，捕获异常时一定要注意将try块内创建的对象清理干净

* MRC下，应该把对象声明在外面，并在finally块中释放

  ```objective-c
  EOCSomeClass *object;
  @try {
      object = [[EOCSomeClass alloc] init];
      [object doSomethingThatMayThrow]; 
  }
  @catch (...) {
  		NSLog(@"Whoops, there was an error. Oh well..."); }
  @finally {
  		[object release];
  }
  ```

* ARC下，其实不会自动处理，因为这需要加入大量代码来跟踪待清理对象，严重影响性能和增加包体积

  可以通过添加-fobjc-arc-exceptions标志开启，但因为OC仅在必须因异常而退出时才抛出异常，默认是不开启的，我们也没有必要开启

  ```objective-c
  @try {
  		EOCSomeClass *object = [[EOCSomeClass alloc] init];
    	[object doSomethingThatMayThrow];
  }
  @catch (...) {
  		NSLog(@"Whoops, there was an error. Oh well...");
  }
  ```



#### No.33: 通过弱引用避免循环引用

* weak修饰符是ARC的特性

* unsafe_unretained vs assign

  语义上是等价的，unsafe_unretained用于对象类型，assign用于基础数据类型

* unsafe_unretained vs weak

  weak修饰的对象在释放时，其指针会自动置nil，而unsafe_unretained仍指向被回收的实例



#### No.34: 使用@autoreleasepool降低内存峰值

* 每个线程默认都有自己的自动释放池，自动释放池在栈空间，会对其括号内包含的对象发送autorelease消息，加入池子里

  每次执行event loop时向自动释放池中的每个对象发送release消息

* 在for循环里使用@autoreleasepool，OS会在新增的自动释放池的块末尾回收某些对象，降低内存峰值

  ```objective-c
  NSArray *databaseRecords = /* ... */;
  NSMutableArray *people = [NSMutableArray new];
  for (NSDictionary *record in databaseRecords) {
    	@autoreleasepool {
    			EOCPerson *person = [[EOCPerson alloc] initWithRecord:record];
    			[people addObject:person];
  		}
  }
  ```

* NSAutoreleasePool vs @autoreleasepool

  @autoreleasepool创建的自动释放池更加轻量，范围更好管理



#### No.35: 用zombie object调试内存管理问题

* 设置NSZombieEnabled，可使OS在回收对象时不将其真正回收，而是转化为zombie object

  实质上是通过method swizzling替换dealloc的实现，修改对象的isa指针，并指向特殊的zombie类，可响应所有selector

* zombie object在响应第一个方法后，打印消息内容和接收者信息，随后终止应用程序



#### No.36: 不要使用retainCount

* retainCount看似有用，但在指定时间点上的retainCount值无法反映对象生命周期全貌

  * 没有考虑到OS稍后会清空自动释放池，下次event loop清空时会发生异常
  * retainCount可能不会返回0，OS有时会优化对象释放行为，在retainCount为1时就回收

  ```objective-c
  while ([object retainCount]) {
  		[object release];
  }
  ```

* 一般调试时使用，但也通常是没有帮助的