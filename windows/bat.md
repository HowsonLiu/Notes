# Windows批处理（cmd/bat）
- 忽略大小写
- 后缀为.bat或者.cmd
- 运行时依次执行每一行的命令
- 每个编写好的批处理文件相当于一个外部命令，将其目录添加至PATH（DOS搜索路径中），即可在任意位置执行
## 变量
- 只能进行整数运算，不支持浮点运算
- 最高精确到32位
### 设置变量与取出变量
- 设置变量:```set a=1```
- 取出变量:```%a%```
注意不要滥加空格，例如```set a = 1```是设置变量```a(空格)```为```(空格)1```
### 字符串拼接
```bat
set str1=nice,
set str2=nice!
set str3=%str1%%str2%
echo %str3%   :: nice,nice!
```
### 字符串截取
**%str:~[m,[,n]]%**
截取str字符m下标长度为n的字符串
```bat
set str1=hello world
echo %str1:~6,5%    :: world
```
### 字符串替换
**%p:str1=str2%**
返回p变量中的str1换成str2的字符串，不改变p
```bat
set p=www.qq.com
echo %p:.=#%    :: www#qq#com
```
### 变量延迟扩展
```bat
setlocal enabledelayedexpansion
set str=test
if %str%==test (
	set str=another test
	echo !str!  :: another test
	echo %str%  :: test
)
```
**扩展**：将用百分号括住的环境变量用其值替换，称之为扩展
在上面的代码中，if语句用括号括住的语句被视为复合语句，把他作为一条语句处理。在执行if语句前，先进行必要的预处理工作：扩展。因此`echo %str%`在扩展后输出test。
使用`setlocal enabledelayedexpansion`延迟扩展后，解释器在遇到复合语句时，并不会作为一条语句同时处理，仍然一条条解释。但是必须要用`!str!`引用变量
## 常用命令
### echo
- echo [message]：在命令行输出message
- echo [on|off]：打开或关闭命令行回显功能（仅在批处理文件中生效，毕竟在CMD中自己打了一遍）
```bat
::2.bat
echo off
date /t
echo on
date /t
```
执行2.bat
```bat
D:\tmp>2.bat

D:\tmp>echo off
08/16/2019 Fri

D:\tmp>date /t
08/16/2019 Fri
```
值得注意的是：echo [on|off]的生效时间为执行之后的下一句。
### @
表示不显示@后面的命令（仅在批处理文件中生效）
### for
在cmd窗口中：`for %i in (command1) do command2`
在bat文件中：`for %%i in (command1) do cammand2`
**注意事项**
- for in do () %%i 不可或缺
- 变量名只能是26个字母，但区分大小写
- command1是容器，放的是待遍历的所有元素，使用空格，tab，逗号，分号或者等号分割
#### for /l
`for /l %%i in (start,step,end) do command`
- start：起始值，step：步长，end：终止值
- 只能是整数
- `for /l %%i in (1,0,1) echo %%i`死循环
- `for /l %%i in (2,1,1) echo %%i`无效语句
- `for /l %%i in (1,2,10) echo %%i`执行至最接近10的数9
#### for /d /r
`for /d %i in (d:\*) do echo %i`
/d用于匹配文件夹，但不能递归，而且不能显示隐藏文件夹
`for /d /r %i in (*) do echo %i`
/r递归
#### for /f
/f命令相当强大，一般用于解析文本。我们先将下面文本保存成1.txt文件
```
1,2:3.4?5
6?7.8:9,10
```
接着执行`for /f %i in (1.txt) do echo %i`，得到的结果是：
```bat
1,2:3.4?5
6?7.8:9,10
```
在这里，容器是1.txt里面的文本，每个元素指的是文件的每一行。
**字符串切分delims=**
执行`for /f "delims=:." %i in (1.txt) do echo %i`，得到的结果是：
```
1,2
6?7
```
delims的作用使用符号列表分割每一个元素，并取第一节。在上例，符号列表是冒号和逗号，第一行被切成了三节，其中第一节是`1,2`，第二行同理。**当不填delims参数时，默认以空格分割**。符号列表可以为空，此时不分割空格
**定点提取tokens=**
执行`for /f "delims=:. tokens=3" %i in (1.txt) do echo %i`，得到的结果是：
```
4?5
9,10
```
delims完成切分之后，可以使用tokens参数制定选取节号（注意从节号1开始，参数间需要添加空格）。若想选取多个节，可以参考如下代码`for /f "delims=:. tokens=2,3" %i in (1.txt) do echo %i %j`，得到的结果是：
```
3 4?5
8 9,10
```
tokens除了可以用逗号分割以外，`tokens=1-3,4`和`tokens=1-4`表示1，2，3，4；`tokens=1,*`表示1，以及剩下的节作为第二节。有多少个节，后面的参数就得顺序新增多少个。例如`tokens=1-3`时，后面为`echo %i %j %k`顺延（很奇怪）
**command1类型**
- 文件名，无需包裹`for /f %i in (1.txt) do echo %i`
- 命令，使用单引号包裹`for /f %i in ('tree') do echo %i`
- 字符串，使用双引号包裹`for /f %i in ("a b c") do echo %i`
### if
常用的三种用法：
- `if %a%==a echo %a% else echo nothing`
- `if exist 1.txt type 1.txt`
- `if errorlevel 1 echo 1 else if errorlevel 2 echo 2 else echo 3`

命令可用括号括住表示复合命令
### choice
根据用户输入返回不同的errorlevel
`choice /c:abc choice a or b or c`
`-c`默认选择Y/N，后面空格后可跟提示，errorlevel返回用户选择的序号（从1开始）
### cls
清屏命令
### rem 和 ::
注释命令
- rem [message]
- ::[message]
### pause
暂停命令，等待用户按任意键后继续
### goto
跳转到标签行，标签前面带:
```bat
:begin
date /t
pause
goto begin
```