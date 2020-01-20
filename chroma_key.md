# 参数
- 图像调整参数
    - opacity 不透明度
    - contrast 对比度
        表示画面从黑到白的渐变层次。越高表示渐变层次越多，从而色彩表现更加丰富。对比度增加，亮的地方更亮，暗的地方更暗
    - brightness 亮度
    - gamma gamma矫正
        - gamma出现的原因
            - 自然界线性增长的亮度与人心理上感受到的均匀灰阶并不是线性关系
                ![](https://pic1.zhimg.com/80/v2-723aa4f5f5e3c6eada193b2955f15096_hd.jpg)
            - 源于CRT显示器的响应曲线，即其亮度与输入电压并不是线性关系
                ![](http://forum.xitek.com/upload/490/49092/49092_1066373688.gif)
        - gamma矫正
            gamma越高，牺牲图像的亮部层次以获取更多暗部层次，通常在比较暗的图像中使用
- chroma参数
    - similarity 相似度
        相似度越低，则图像中只有与key color完全吻合的像素才会被扣掉
    - smoothness 平滑度
        我理解为类似于羽化的效果，减少抠图产生的锯齿？
    - key color spill reduction 减少key color颜色溢出
        color spill是颜色溢出。指的是在绿幕环境下进行拍摄，物体上会反射到绿色的自然光。因此key color spill reduction参数用于减少这部分反射光的影响。

# obs中的颜色比较
由于`similarity`参数的存在，因此如何度量两个像素点之间颜色的差异是一个问题。在obs中，使用了如下的方法进行度量：
1. **将rgba值转化为yuva值**
    在obs中，引入了yuv_mat的常量(来源无从考量...)
    ```
    static const float g_YuvMat[16] = { 0.182586f, -0.100644f, 0.439216f, 0.0f,
					0.614231f, -0.338572f, -0.398942f, 0.0f,
					0.062007f, 0.439216f, -0.040274f, 0.0f,
					0.062745f, 0.501961f, 0.501961f, 1.0f };
    ```
    接着利用矩阵相乘，类似于游戏中的坐标系变换，将1x4的rgba数据叉乘4x4的yuv坐标系可以得到1x4的yuva数据
2. **提取uv分量**
    根据yuva数据的定义可知，a同样是alpha通道，比较颜色不需要，而y是指亮度，也不需要，因此只需要uv分量就能够进行颜色比较
3. **量化uv分量差异**
    差异的计算留到hlsl中计算，使用hlsl中的`distance`函数得到key_color与此时像素中color的差异值

# 细说rgba转化成yuva
在项目中，有两段rgba转化成yuva的代码。第一段是在设置chroma参数的时候，将key_color的rgba转为yuva，并将其uv分量通过寄存器传到hlsl中；第二段是在GPU的并行运算中，并行计算每个像素点，将每个像素点的rgba数据转换成uv分量进行比较差异。
## CPU中的转换(C++)
```c++
uint32_t key_color = m_uKeyColor;
vec4_from_rgba(&key_rgb, key_color);
memcpy(&yuv_mat_m4, g_YuvMat, sizeof(g_YuvMat));
vec4_transform(&key_color_v4, &key_rgb, &yuv_mat_m4);	
XMFLOAT2 keyColorUV = XMFLOAT2(key_color_v4.y, key_color_v4.z);
```
这是转换的相关代码
```c++
static inline void vec4_from_rgba(struct vec4 *dst, uint32_t rgba)
{
    dst->x = (float)((double)(rgba & 0xFF) * (1.0 / 255.0));
    rgba >>= 8;
    dst->y = (float)((double)(rgba & 0xFF) * (1.0 / 255.0));
    rgba >>= 8;
    dst->z = (float)((double)(rgba & 0xFF) * (1.0 / 255.0));
    rgba >>= 8;
    dst->w = (float)((double)(rgba & 0xFF) * (1.0 / 255.0));
}
```
从这个32位int类型的rgba数据转换成vec4类型的函数可以看出，在obs中，为了在这个转换中优化性能，32位的rgba数据是以逆序的方式存放的，也就是说int的高位字节存放的不是r，反而是a。这个是我遇到的第一个坑。
```c++
static inline void vec4_transform(struct vec4 *dst, const struct vec4 *v,
    const struct matrix4 *m)
{
    struct vec4 temp;
    struct matrix4 transpose;

    matrix4_transpose(&transpose, m);

    temp.x = vec4_dot(&transpose.x, v);
    temp.y = vec4_dot(&transpose.y, v);
    temp.z = vec4_dot(&transpose.z, v);
    temp.w = vec4_dot(&transpose.t, v);

    vec4_copy(dst, &temp);
}
```
这里就是将rgba通过yuv矩阵转换成yuva数据的代码了。输入参数是1x4的v以及4x4的m，输出结果是1x4的dst。我们可以看到他对yuv矩阵进行了一次转置，转置的原因是为了能用点乘实现叉乘。由于输入参数是1x4的v，因此$dst(1, 1) = \sum_1^4 v(1, i) * mat(i, 1)$，在对mat进行转置之后，mat的行就变为了列，即$dst(1, 1) = \sum_1^4 v(1, i) * mat(1, i)$，这样子性能又得到了进一步的优化了。
## GPU中的转换(effect)
```hlsl
uniform float4x4 yuv_mat = { 0.182586,  0.614231,  0.062007, 0.062745,
                            -0.100644, -0.338572,  0.439216, 0.501961,
                             0.439216, -0.398942, -0.040274, 0.501961,
                             0.000000,  0.000000,  0.000000, 1.000000};

float GetChromaDist(float3 rgb)
{
	float4 yuvx = mul(float4(rgb.rgb, 1.0), yuv_mat);
	return distance(chroma_key, yuvx.yz);
}
```
这是obs中的像素着色器代码，他对每个像素进行运算返回相似度。值得注意的是，这里的yuv_mat跟C++代码中的yuv_mat互逆。他使用了一个mul函数，进行矩阵相乘。就在这里，我遇到了最大的坑。
## GPU中的转换(HLSL)
遇到的问题是这样子的：代码能跑通不报错，输入图像跟输出图像都正常，而且也存在抠图的功能，但关键是抠出的颜色与key_color不一致。比如说我想抠绿色(0xFF00FF00)，但实际抠的颜色却不是绿色，而且两者之间并无规律可循。经过一段时间的控制变量排除原因，我最终将异常代码锁定在`GetChromaDist`函数里面。最终，我把代码修改成这样
```hlsl
float GetChromaDist(float3 rgb)
{
	float4x4 yuv_mat = { 0.182586, 0.614231, 0.062007, 0.062745,
			-0.100644, -0.338572, 0.439216, 0.501961,
			0.439216, -0.398942, -0.040274, 0.501961,
			0.000000, 0.000000, 0.000000, 1.000000 };
    float4 yuvx = mul(yuv_mat, float4(rgb.rgb, 1.0));
    return distance(chroma_key, yuvx.yz);
}
```
### 改动1：yuv_mat从全局变量变成局部变量
这个点很奇怪，hlsl也是支持全局变量的，但是全局变量初值来源于C++，在hlsl中赋初值貌似不行，因此我把它移到局部变量去了
### 改动2：mul参数位置互换
