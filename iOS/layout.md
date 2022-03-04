# 布局技术分类
- nib：nib是NeXT interface builder的英文缩写，以二进制的形式存储界面信息，是IB3.0以前的文件格式。
- xib：xib是xml interface builder的英文缩写，是IB3.0之后苹果公司推出的新一代，以xml格式存储界面信息，在最终执行前，xib文件会被编译为nib文件。
- storyboard：故事版文件，是苹果最新推出的用于在界面开发中替代xib文件的一种新技术。本质上是一个xml文件的集中管理区，不但可以描述xib单个界面的结构，还可以描述界面之间的跳转及依赖关系。主要是靠手拖，感觉像积木玩具。
- frame：等效于代码版的storyboard，但更灵活。目前比较常用。如果没有好的适配方案，是多众多机型尺寸还是有点棘手。
- AutoLayout：自动布局（AutoLayout）是iOS6发布的界面布局技术，该算法的主要思想是：将基于约束系统的布局规则（本质上是表示视图布局关系的线性方程组）转化为表示规则的视图几何参数。实际上AutoLayout算法本身并非有苹果发明，只是苹果用Objective-C去实现了该算法，方便iOS开发者使用。AutoLayout有多种使用方式，如可视化工具：Xcode的Interface Builder，纯代码：以Masonry为代表。
- FlexBox：弹性布局（Flexible Box）。对，就是目前Web端最流行的布局方式（以前是盒子模型），现在APP上也能使用。此方案就扩展出很多技术，如Yoga（最牛逼的代表，Facebook出品，衍生出很多上层方案，如跨平台的ReactNative、Android的Litho、iOS的Yogakit）、FlexboxLayout（Android代表，Google出品）、FlexLib、FLEX等。
- swiftUI：官网，苹果官方推荐。


# AutoLayout
AutoLayout 实现了名为 [Cassowary](https://en.wikipedia.org/wiki/Cassowary_(software)) 的布局算法。用户可以对各个控件进行约束，该算法会自动计算出满足所有约束的布局效果
## 约束
- 约束的对象
    - Super View
    - Safe Area
    - 同一层级的其他View

- 约束的规则
    - Leading, Tailing
    - centerX, centerY
    - Left, Right, Top, Bottom
    - Width, Height

- 约束的优先级（0~1000）
    - Low (250)
    - High (750)
    - Required (1000)

## 坐标系
坐标系的原点位于屏幕的左上角

## 