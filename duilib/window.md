# duilib窗口
## 模态
duilib通过窗口禁用函数以及消息循环来模拟模态窗口的实现
```c++
UINT Window::ShowModal()
{
    ASSERT(::IsWindow(m_hWnd));
    UINT nRet = 0;
    HWND hWndParent = GetWindowOwner(m_hWnd);
    ::ShowWindow(m_hWnd, SW_SHOWNORMAL);
    ::EnableWindow(hWndParent, FALSE);      // 开始模态，禁用父窗口
    MSG msg = { 0 };
    HWND hTempWnd = m_hWnd;
    while( ::IsWindow(hTempWnd) && ::GetMessage(&msg, NULL, 0, 0) ) {   // 这里跑消息循环
        if( msg.message == WM_CLOSE && msg.hwnd == m_hWnd ) {
            nRet = msg.wParam;
            ::EnableWindow(hWndParent, TRUE);
            ::SetFocus(hWndParent);
        }
        if( !GlobalManager::TranslateMessage(&msg) ) {      // 这里分发消息
            ::TranslateMessage(&msg);
            ::DispatchMessage(&msg);
        }
        if( msg.message == WM_QUIT ) break;
    }
    ::EnableWindow(hWndParent, TRUE);       // 结束模态，启用父窗口
    ::SetFocus(hWndParent);
    if( msg.message == WM_QUIT ) ::PostQuitMessage(msg.wParam);
    return nRet;
}
```
`Window::ShowModal`使得此窗口成为父窗口的模态窗口。

调用此函数后，父窗口将被禁用，然后在此函数里运行新的消息循环（原来那个就被阻塞了），直至新的消息循环结束，才启用父窗口

新的消息循环里主要是做3件事
- 从此窗口的`WM_CLOSE`消息里获取结束码
- 分发其他消息
- 处理`WM_QUIT`消息

