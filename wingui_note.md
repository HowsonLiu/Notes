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
    if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))     // 前面他加载了一个加速器
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