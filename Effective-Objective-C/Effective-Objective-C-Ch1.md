### 第一章: 熟悉Objective-C

#### No.1: 了解Objective-C语言的起源

* C -> Smalltalk -> Objective-C

* 消息结构 vs 函数调用

  * OC是使用消息结构的语言，运行时所执行的代码由runtime决定
* C和C++是使用函数调用的语言，运行时所执行的代码由编译器决定，如果函数是多态的，则需要在运行时去virtual table查找



#### No.2: 在类的头文件中尽量少引入其他文件

* 若ClassA持有属性ClassB，则使用@class在ClassA.h中@class ClassB，而不用#import "ClassB.h"

  在ClassA.m中#import "ClassB.h"，把引入ClassB.h的时机尽量延后，有效提高编译效率

* 若ClassA.h和ClassB.h需要互相#import，则需要使用@class解决循环引用

  协议应该单独声明在一个头文件里，防止产生潜在的循环引用

* @class不能用做声明ClassA的超类和协议，因为此时编译器需要知道其中定义的方法



#### No.3: 多用字面量语法，少用与之等价的方法

* 应该使用字面量语法，提高代码的可读性

* 使用NSArray时需要注意nil值

  ```objective-c
  id object1 = /* ... */;
  id object2 = /* ... */;
  id object3 = /* ... */;
  // when object2 = nil
  NSArray *arrayA = [NSArray arrayWithObjects: object1, object2, object3, nil]; // [object1]
  NSArray *arrayB = @[object1, object2, object3]; // 抛出异常
  ```

* 使用NSDictionary时需要注意dictionaryWithObjectsAndKeys:方法是value-key结构，应该使用key-value结构的字面量语法

  NSDictionary在遇到nil值时和NSArray一样，会抛出异常

  ```objective-c
  // value-key
  NSDictionary *personData = [NSDictionary dictionaryWithObjectsAndKeys:
  @"Matt", @"firstName",
  @"Galloway", @"lastName",
  [NSNumber numberWithInt:28], @"age",
  nil];
  
  // key-value
  NSDictionary *personData = @{
    @"firstName" : @"Matt",
  	@"lastName" : @"Galloway", 
    @"age" : @28
  };
  ```

* 使用字面量创建的NSString、NSArray和NSDictionary都是不可变的，需要手动mutableCopy

  ```objective-c
  NSMutableArray *mutable = [@[@1, @2, @3, @4, @5] mutableCopy];
  ```



#### No.4: 多用类型常量，少用#define预处理指令

* 应该使用类型常量替换#define，这样可以带上类型信息

  ```objective-c
  #define ANIMATION_DURATION 0.3
  
  static const NSTimeInterval kAnimationDuration = 0.3;
  ```

* 慎重在头文件里声明常量，OC没有命名空间，每个#import该头文件的文件中，都会出现我们声明的常量

  如果只是为了在.m里使用，可以在.m的@implementation外声明

* 一定要用static和const修饰符

  若有试图修改常量的操作，const修饰符会让编译器报错

  若有多个.m声明同一个常量，static会限制每个.m里声明的常量仅在该.m里可见，不为常量创建external symbol

* 用extern创建外界可见的常量，此类常量存放在全局符号表中

  ```objective-c
  // In the header file
  extern NSString *const EOCStringConstant;
  
  // In the implementation file
  NSString *const EOCStringConstant = @"VALUE";
  ```



#### No.5: 用枚举表示状态、选项和状态码

* 默认的enum不支持指定底层数据类型，编译器会根据枚举值数量自动选择底层数据类型

  我们应该使用宏定义好的NS_ENUM和NS_OPTIONS，分别对应单选和多选，支持我们自定义底层数据类型

  ```objective-c
  typedef NS_ENUM(NSUInteger, EOCConnectionState) {
  		EOCConnectionStateDisconnected,
  		EOCConnectionStateConnecting,
  		EOCConnectionStateConnected,
  };
  
  typedef NS_OPTIONS(NSUInteger, EOCPermittedDirection) {
  		EOCPermittedDirectionUp = 1 << 0,
  		EOCPermittedDirectionDown = 1 << 1,
  		EOCPermittedDirectionLeft =1<<2,
  		EOCPermittedDirectionRight = 1 << 3,
  };
  ```

* 使用switch处理枚举时不要实现default分支，这样以后新加入枚举值时编译器会有提示