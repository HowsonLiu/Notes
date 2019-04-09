# Windows窗口编程
基本流程：
1. 定义、注册窗口类
2. 初始化窗口
    1. 创建窗口
    2. 显示和更新窗口
3. 创建消息循环
4. 终止应用程序
5. 窗口过程
6. 处理消息
## 主函数
```c++
int APIENTRY wWinMain(_In_ HINSTANCE hInstance,
                     _In_opt_ HINSTANCE hPrevInstance,
                     _In_ LPWSTR    lpCmdLine,
                     _In_ int       nCmdShow)
{
    UNREFERENCED_PARAMETER(hPrevInstance);
    UNREFERENCED_PARAMETER(lpCmdLine);

    // TODO: Place code here.

    // Initialize global strings
    LoadStringW(hInstance, IDS_APP_TITLE, szTitle, MAX_LOADSTRING);
    LoadStringW(hInstance, IDC_WINDOWSLEARNING, szWindowClass, MAX_LOADSTRING);
    MyRegisterClass(hInstance);                 // 定义并注册窗口类

    // Perform application initialization:
    if (!InitInstance (hInstance, nCmdShow))    // 初始化窗口
    {
        return FALSE;
    }

    HACCEL hAccelTable = LoadAccelerators(hInstance, MAKEINTRESOURCE(IDC_WINDOWSLEARNING));

    MSG msg;

    // Main message loop:
    while (GetMessage(&msg, nullptr, 0, 0))
    {
        if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
        {
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
    }

    return (int) msg.wParam;
}
```

## 注册窗口
它是通过下面这个全局函数帮助我们注册窗口
```c++
ATOM                MyRegisterClass(HINSTANCE hInstance);
//
//  FUNCTION: MyRegisterClass()
//
//  PURPOSE: Registers the window class.
//
ATOM MyRegisterClass(HINSTANCE hInstance)
{
    WNDCLASSEXW wcex;

    wcex.cbSize = sizeof(WNDCLASSEX);

    wcex.style          = CS_HREDRAW | CS_VREDRAW;      // 窗口类风格
    wcex.lpfnWndProc    = WndProc;      // 窗口过程函数
    wcex.cbClsExtra     = 0;            // 类附加数据
    wcex.cbWndExtra     = 0;            // 窗口附加数据
    wcex.hInstance      = hInstance;    // 窗口类的实例句柄
    wcex.hIcon          = LoadIcon(hInstance, MAKEINTRESOURCE(IDI_WINDOWSLEARNING));    // 图标
    wcex.hCursor        = LoadCursor(nullptr, IDC_ARROW);   // 鼠标光标
    wcex.hbrBackground  = (HBRUSH)(COLOR_WINDOW+1);     // 着色窗口背景的刷子
    wcex.lpszMenuName   = MAKEINTRESOURCEW(IDC_WINDOWSLEARNING);    // 菜单名
    wcex.lpszClassName  = szWindowClass;        // 窗口类名
    wcex.hIconSm        = LoadIcon(wcex.hInstance, MAKEINTRESOURCE(IDI_SMALL));

    return RegisterClassExW(&wcex);
}
```
这个全局函数的作用就是实例化一个`WNDCLASSEXW`，通过修改他的成员变量定义出我们的窗口类，最后调用`RegisterClassExW`函数向Windows注册。
## 初始化窗口
初始化窗口`InitInstance`的定义如下：
```c++
//
//   FUNCTION: InitInstance(HINSTANCE, int)
//
//   PURPOSE: Saves instance handle and creates main window
//
//   COMMENTS:
//
//        In this function, we save the instance handle in a global variable and
//        create and display the main program window.
//
BOOL InitInstance(HINSTANCE hInstance, int nCmdShow)
{
   hInst = hInstance; // Store instance handle in our global variable
    // 创建窗口
   HWND hWnd = CreateWindowW(szWindowClass, szTitle, WS_OVERLAPPEDWINDOW,
      CW_USEDEFAULT, 0, CW_USEDEFAULT, 0, nullptr, nullptr, hInstance, nullptr);

   if (!hWnd)
   {
      return FALSE;
   }

   ShowWindow(hWnd, nCmdShow);  // 显示窗口
   UpdateWindow(hWnd);          // 更新窗口

   return TRUE;
}
```
### 创建窗口
```c++
HWND CreateWindow(
  LPCTSTR lpClassName,  // 刚刚上面注册的窗口类名
  LPCTSTR lpWindowName, // 窗口标题
  DWORD dwStyle,        // 风格比如最大化最小化
  int x,                // 下面4个都是窗口的位置大小信息
  int y,                 
  int nWidth,            
  int nHeight,          
  HWND hWndParent,      // 父窗口
  HMENU hMenu,          // 菜单
  HANDLE hInstance,     // 应用实例名
  LPVOID lpParam        // 附加数据
);
```
### 显示和更新窗口
显示窗口
```c++
ShowWindow(hWnd, nCmdShow);     // 第一个参数是窗口句柄，第二个参数是显示窗口的方式比如最大化
```
更新窗口
```c++
UpdateWindow(hWnd);
```
更新窗口会产生一个`WM_PAINT`消息，用于刷新窗口
## 消息循环
### GetMessage
```c++
MSG msg;
// Main message loop:
while (GetMessage(&msg, nullptr, 0, 0))
{
    if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))     // 将快捷键变成对应菜单命令
    {
        TranslateMessage(&msg); // 翻译。比如取WM_KEYDOWN和WM_KEYUP转化成WM_CHAR
        DispatchMessage(&msg);  // 使用窗口过程函数分发消息
    }
}
```
`GetMessage(&msg, nullptr, 0, 0)`函数第二个参数是窗口句柄，nullptr表示接受该应用程序所有窗口的消息，第三第四个参数是消息范围，一般后三个取默认值。当消息是`WM_QUIT`时返回False，否则返回True
### PeekMessage
```c++
while (TRUE)       
{        
    if (PeekMessage (&msg, NULL, 0, 0, PM_REMOVE))        
    {        
        if (msg.message == WM_QUIT)        
               break ;        
        TranslateMessage (&msg) ;        
        DispatchMessage (&msg) ;        
    }        
    else             
            // 完成某些工作的其它行程序        
}       
```
### GetMessage和PeekMessage的区别
- `GetMessage`每次从消息队列取出并删除消息，仅当消息是`WM_QUIT`才返回False；而`PeekMessage`像是窥探消息队列，有消息就返回True，没消息返回False
- `GetMessage`取不到消息会阻塞，`PeekMessage`不会
## 终止应用程序
一旦Windows进入消息循环，终止的唯一办法是使用`PostQuitMessage(int nExitCode)`。这个函数表明有个线程请求终止，一般在主窗口收到`WM_DESTROY`消息的时候会发送`WM_QUIT`
## 窗口过程函数`WndProc`
`WndProc`是一个全局函数
```c++
LRESULT CALLBACK    WndProc(HWND, UINT, WPARAM, LPARAM);
//
//  FUNCTION: WndProc(HWND, UINT, WPARAM, LPARAM)
//
//  PURPOSE: Processes messages for the main window.
//
//  WM_COMMAND  - process the application menu
//  WM_PAINT    - Paint the main window
//  WM_DESTROY  - post a quit message and return
//
//
LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    switch (message)
    {
    case WM_COMMAND:
        {
            int wmId = LOWORD(wParam);
            // Parse the menu selections:
            switch (wmId)
            {
            case IDM_ABOUT:
                DialogBox(hInst, MAKEINTRESOURCE(IDD_ABOUTBOX), hWnd, About);
                break;
            case IDM_EXIT:
                DestroyWindow(hWnd);
                break;
            default:
                return DefWindowProc(hWnd, message, wParam, lParam);
            }
        }
        break;
    case WM_PAINT:
        {
            PAINTSTRUCT ps;
            HDC hdc = BeginPaint(hWnd, &ps);
            // TODO: Add any drawing code that uses hdc here...
            EndPaint(hWnd, &ps);
        }
        break;
    case WM_DESTROY:
        PostQuitMessage(0);     // 发送WM_QUIT
        break;
    default:
        return DefWindowProc(hWnd, message, wParam, lParam);
    }
    return 0;
}
```
窗口处理函数实际上是一个巨大的switch语句，里面对收到的各种消息进行处理
## SendMessage和PostMessage的区别
- `SendMessage`要等消息处理完才返回，`PostMessage`立即返回
- 实际上`SendMessage`是直接调用目标窗口的窗口过程函数`WndProc`,而`PostMessage`将消息放入消息队列后返回
# Windows消息机制
## 消息的定义
```c++
typedef struct tagMSG
{
    HWND hwnd;      // 窗口句柄
    UINT message;   // 消息号
    WPARAM wParam;  // 参数1
    LPARAM lParam;  // 参数2
    DWORD time;     // 消息创建时的消息
    POINT pt;       // 消息创建时光标位置
} 	MSG;
```
## 消息的类型
- 系统定义的消息
    - 窗口消息，比如绘制窗口
    - 命令消息，`WM_COMMAND`
    - 通知消息，`WM_NOTIFY`
- 应用定义的消息
    - 用户自定义的消息，`WM_USER`
    - 程序之间通信的消息，`WM_APP`
## 消息队列
- 系统消息队列   
操作系统唯一的消息队列，接受驱动传来的消息，并把消息放进目标窗口所在的线程消息队列中等待处理
- 线程消息队列   
一个线程被创建出来默认不是GUI线程，当他调用GDI函数时，Windows才将它转换成GUI线程。每个GUI线程关联一个THREADINFO结构，这个结构里面有4个消息队列：
    - 发送消息队列：保存其他线程通过`SendMessage`发送给他的消息
    - 登记消息队列：保存其他队列通过`PostMessage`发送给他的消息
    - 输入消息队列：保存系统队列分发过来的消息
    - 响应消息队列：保存发送消息后的结果。比如`SendMessage`成功后会收到一个Reply消息（里面是返回值）以唤醒自己

`WM_PAINT`和`WM_TIMER`只有在消息队列没有其他消息时才会被处理，`WM_PAINT`还会合并以提高效率，其余消息按照先进先出被处理。消息队列有容量，超过不会处理
## 发送消息   

|  |SendMessage|PostMessage|
|:--:|:--:|:--:|
|相同线程|直接调用窗口处理函数|把消息放进队列|
|不同线程|发送后收到Reply前阻塞|最好使用`PostThreadMessage`代替|

在`SendMessage`会给目标线程添加QS_SENDMESSAGE标记，接着自己进入idle状态，此时若收到别人的`SendMessage`，则目标线程立即处理

## 接收消息的算法
1. QS_SENDMESSAGE标记，发送消息队列为空时才清除
2. 登记消息队列，优先级越高越快响应
3. QS_QUIT
4. 输入消息队列
5. QS_PAINT，返回WM_PAINT
6. QS_TIMER，返回WM_TIMER

## SendMessage死锁解决
- `PostMessage`
- `SendMessageTimeout`
- `SendNotifyMessage`