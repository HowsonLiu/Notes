# Swift快速上手
# 变量的声明  
- `var` 变量  
- `let` 常量  

声明的格式
```
let constName = <initial value>
let varibleName:<data type> = <optional initial value>
```
# Optionals
Optional是一个泛型，表示该值可以为空
变量后加`!`表示解引用，否则访问的是他本身
```swift
var str: Optional<String>
var str: String?            // 等同上一句话
print(str)                  // nil
str = "Hello"
print(str)                  // Optional("Hello")
print(str!)                 // "Hello"
```

使用可选绑定机制，避免显式强制解包
```swift
if let str = str {
    print(str)
}
else {
    print("Nil")
}
```

使用`??`提供默认值，`??`语句属于短路求值
```swift
let res = str??"123"
```

使用`guard`处理异常情况
```swift
func addChar(str: String?) -> String? {
    guard let str = str else {return nil}
    return str+"\n"
}
```

# 访问控制
- private  
    修饰的属性或者方法只能在当前类访问
- fileprivate (swift 3)  
    修饰的属性或者方法只能在当前文件中访问，同一文件中的其他类也能访问
- internal  
    默认访问级别。修饰的属性或者方法可以在源代码的整个模块中访问
- public  
    可以被任何代码访问。但在模块外不可以被override和继承，模块内可以
- open
    模块外可以被override和继承