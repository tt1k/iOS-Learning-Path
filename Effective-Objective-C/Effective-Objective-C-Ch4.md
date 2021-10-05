### 第四章: 协议与分类

#### No.23: 通过委托和数据源协议进行对象间通信

* delegate通过定义一套接口，实现不同hierachy层级的对象间的通信

  数据源协议本质上也是一种专门用于传输数据而不执行动作的delegate

* delegate应该使用weak属性，避免循环引用

* 可以实现含有位段段结构体，缓存delegate对象是否能响应其定义的接口

  ```objective-c
  @interface EOCNetworkFetcher () {
    	struct {
  				unsigned int didReceiveData : 1;
        	unsigned int didFailWithError : 1;
        	unsigned int didUpdateProgressTo : 1;
  		} _delegateFlags;
  }
  @end
  
  - (void)setDelegate:(id<EOCNetworkFetcher>)delegate {
    	_delegate = delegate;
  		_delegateFlags.didReceiveData = [delegate respondsToSelector:@selector(networkFetcher:didReceiveData:)];
  		_delegateFlags.didFailWithError = [delegate respondsToSelector:@selector(networkFetcher:didFailWithError:)];
    	_delegateFlags.didUpdateProgressTo = [delegate respondsToSelector:@selector(networkFetcher:didUpdateProgressTo:)];
  }
  
  if (_delegateFlags.didUpdateProgressTo) {
    	[_delegate networkFetcher:self didUpdateProgressTo:currentProgress];
  }
  ```



#### No.24: 将类的实现代码分散到便于管理的数个分类之中

* 使用Category根据代码逻辑合理拆分.m文件，debug时能看到对应的Category而快速定位

* 应该把私有方法归入Private分类中，隐藏实现细节

  使用者在backtrace中看到Private分类标识时能知道不应该调用此方法



#### No.25: 总是为第三方类的分类名称添加前缀

* 向第三方分类中添加分类时，应该给其名称和方法加上自己的专用前缀，避免因重名而出现的bug



#### No.26: 勿在分类中声明属性

* 虽然可以通过关联对象为Category添加属性，但这样需要写runtime的方法调用，且在内存管理上容易出错

* 我们应该把属性定义在头文件中，Category的目标在于扩展类的功能，而非封装数据

* readonly属性在有必要时可以直接加在Category内

  ```objective-c
  @interface NSCalendar (EOC_Additions)
  @property (nonatomic, strong, readonly) NSArray *eoc_allMonths;
  @end
  
  @implementation NSCalendar (EOC_Additions)
  -(NSArray*)eoc_allMonths {
  		return @[@"January", @"February", @"March", @"April", @"May", @"June", @"July", @"August", @"September", @"October", @"November", @"December"];
  }
  @end
  ```



#### No.27: 使用Extension隐藏实现细节

* Extension是唯一可以新增属性的特殊Category

  在Extension中可以新增属性，可以override头文件里声明的属性，可以遵从不想被外界知道的协议



#### No.28: 通过协议提供匿名对象

* 遵从某个指定协议的delegate通常声明为id类型，因此是匿名的，其实并没有什么太大的用处

  ```objective-c
  @property (nonatomic, weak) id <EOCDelegate> delegate;
  ```

* NSDictionary通过匿名对象设置key的内存管理语义为copy，value为retain

  ```objective-c
  -(void)setObject:(id)object forKey:(id<NSCopying>)key;
  ```