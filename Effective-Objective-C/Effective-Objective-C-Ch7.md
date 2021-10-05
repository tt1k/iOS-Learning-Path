### 第七章: 系统框架

#### No.47: 熟悉系统框架

* 无缝桥接(toll-free bridging)

  可以把Foundation和CoreFoundation框架中的对象互相转换，例如NSString和CFString

* 优秀的OC开发者应该掌握C的核心概念



#### No.48: 多用block-based枚举，少用for循环

* for -> NSEnumerator -> for in -> enumerateObjectsUsingBlock:

  使用block-based枚举可以获得idx这样的参数信息，更加方便，而且本身可通过GCD实现遍历，是其他方式无法轻易实现的

  ```objective-c
  NSArray *anArray = /* ... */;
  [anArray enumerateObjectsUsingBlock:^(id object, NSUInteger idx, BOOL *stop) {
    	// Do something with 'object'
      if (shouldStop) {
  		    *stop = YES;
      }
  }];
  ```

* 若知道待遍历的collection包含的对象类型，则应该修改block签名，增加代码的可维护性



#### No.49: 对自定义其内存管理语义的collection使用toll-free bridging

* 一个简单的toll-free bridging

  __bridge表示ARC仍负责该对象的release

  __bridge_retained表示ARC交出该对象的所有权，由我们手动CFRelease释放内存

  CoreFoundation到Foundation使用__bridge_transfer

  ```objective-c
  NSArray *anNSArray = @[@1, @2, @3, @4, @5];
  CFArrayRef aCFArray = (__bridge CFArrayRef)anNSArray;
  NSLog(@"Size of array = %li", CFArrayGetCount(aCFArray));
  // Output: Size of array = 5
  ```

* CoreFoundation创建collection，支持自定义回调函数，然后可通过toll-free bridging转成指定内存管理语义的OC collection



#### No.50: 构建缓存时选用NSCache而不是NSDictionary

* NSCache提供优雅的自动删减功能，且线程安全，并支持设置缓存的上限值

  NSCache不会copy键，这样可以让不支持NSCopying的对象作为键

  NSDictionary的键必须支持NSCopying

* 使用NSPurgeableData搭配NSCache，可实现自动清除数据的功能

* 只有重新获得或计算很耗时的对象才值得被放入缓存



#### No.51: 精简initialize和load的实现

* load vs initialize

  |          | load                                       | initialize                       |
  | -------- | ------------------------------------------ | -------------------------------- |
  | 调用时机 | APP启动前                                  | 对象第一次收到消息时             |
  | 覆写机制 | 不支持覆写机制，类的load比分类的load先执行 | 支持覆写，需要在方法体里判断类型 |

* 两个方法里都不应该执行太多操作，避免程序响应慢，减少发送循环引用的概率

* 无法在编译期设定的全局常量，可以在initialize里初始化



#### No.52: NSTimer会保留其target

* NSTimer会保留其target，直到NSTimer被invalidate

  一次性的NSTimer执行完任务后直接invalidate

* repeating NSTimer很容易发送循环应用，使用时需要格外小心

  通过加入block扩充NSTimer功能，可有效减少循环引用的概率