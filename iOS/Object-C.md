# 快速上手
- `#import` 解决头文件重复包含，也就是对应VC中的`#program once`  
- `NS`前缀的函数，表示函数来自于Cocoa
- `@"Hello"` 字符串前面加上`@`，表示字符串为`NSString`
- 字符串格式化  
    ```@"String is %@"```  
    `%@`表示取出对象中的description方法  
- `@class` 表示声明一个类，不需要引用头文件，常用于指针的声明

# OOP
## 类的声明
```ObjC
/* @interface 表示继承 */
@interface Circle : NSObject
{
@private
ShapeColor fillColor;
ShapeRect bounds;
}

/* (return) functionName: (Type1)Arg1 */
/* - 在前表示成员函数 */
- (void) draw;
- (void) setFillColor: (ShapeColor) fillColor;
- (void) setBounds: (ShapeColor) fillColor;
/* + 在前表示静态函数 */
+ (void) createCircle;
/* @end完成对Circle类的声明 */
@end
```
## 类的实现
```ObjC
/* @implementation 表示类函数实现 */
@implementation Circle
- (void) setFillColor: (ShapeColor) c
{
    /* self = this */
    self->fillColor = c;
}
```
## 实例化
```ObjC
@interface Car : NSObject
{
    Engine* engine;
    Tire* tires[4];
}
@end

@implementation Car
/* 构造函数 */
- (id) init
{
    if(self = [super init]) {
        engine = [Engine new];
        for(int i = 0; i < 4; ++i)
            tires[i] = [Tire new];
    }
    return (self);
}
```


# 属性
在Objective-C 2.0中引入  
- 使用`@property`声明一个对象的属性  
- 使用`@synthesize`创建属性的访问代码  
- 使用`@dynamic`让编译器不生成访问代码及实例变量

## `@property`
关于声明位置
- 在头文件声明，子类可以访问属性
- 在.m文件声明，子类不可访问属性  

关于属性扩展
- 线程相关
    - `nonatomic`：非原子性，提高性能
    - `atomic`：原子性，线程安全（默认）
- 读写相关
    - `readwrite`：同时生成getter和setter
    - `readonly`：只生成getter
    - `writeonly`：只生成setter
- 浅拷贝
    - `retain`：引用计数+1
    - `assign`：不涉及引用计数
    - `weak`：不涉及引用计数，目标销毁时自动置nil
- 深拷贝
    - `copy`/`strong`：深拷贝对象，新对象的引用计数为1

## `@synthesize`
让编译器自动生成getter/setter方法。  
当`@synthesize`跟`@dynamic`都没写时，默认生成`@synthesize var = _var`  

## `@dynamic`
拒绝编译器实现getter/setter方法
```Objc
@property (readonly) float bodyMessIndex;
@dynamic bodyMessIndex;
- (float) bodyMessIndex
{
    // my code
}
```

# 类别/类扩展/继承
## 类别
Objective-C支持为现有的类添加一些方法（即使没有类的源代码）。主要用途为
- 将类的代码分散到不同文件以及框架
- 创建对私有方法的前向引用
- 向对象添加非正式协议

缺点
- 不能添加变量
- 函数名冲突，会直接取代原函数

优点
- 子类可以继承新添加的方法  

## 类扩展
能为某个类附加额外属性、变量、和方法。但只能是私有的。需要源代码。

缺点
- 附加的只能是`@private`
- 只能在源文件中使用
- 不能被子类继承