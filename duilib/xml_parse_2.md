# 网易duilib中的xml解析（二）WindowBuilder
[上文](xml_parse.md)介绍了网易duilib是如何通过`CMarkup`支持标准xml语法的，本文主要介绍duilib特化

# xml的作用
在duilib中，xml主要有两种类型：
- 全局资源描述
- 窗口样式设计

## 全局资源描述
在**程序目录下themes\default\global.xml**文件里，定义了全局统一的资源类型。其根节点必须为`<Global>`标签，如
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Global>
    <!-- 字体 -->
    <FontResource file="FZBangSXJW.TTF" name="方正榜书行简体"/>
    <!--字体大小-->
    <Font name="system" size="10" />
    <!--颜色-->
    <TextColor name="white30" value="#bbffffff" />
    <!--样式-->
    <Class name="font_title" value="font=&quot;2&quot;" />
</Global>
```
这些资源可以通过名字在任何地方被使用

全局资源在静态成员函数`GlobalManager::LoadGlobalResource`中被加载
```c++
void GlobalManager::LoadGlobalResource()
{
	ui::WindowBuilder dialog_builder;
	ui::Window paint_manager;
	dialog_builder.Create(L"global.xml", CreateControlCallback(), &paint_manager);
}
```

## 窗口样式设计
窗口xml文件的根节点必须为`<Window>`标签，文件可以放在任何地方
```xml
<?xml version="1.0" encoding="utf-8"?>
<Window size="100,100" caption="0,0,0,48">
    <Box>
        <Label text="hello"/>
    </Box>
</Window>
```
窗口类必须继承自`ui::WindowImplBase`，并且重写其`GetSkinFolder`以及`GetSkinFile`方法设置xml文件

窗口在初始化的时候，会加载xml文件
```c++
LRESULT WindowImplBase::OnCreate(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled)
{
    // ...

    WindowBuilder builder;
    Box* pRoot=NULL;
    if (GetResourceType()==UILIB_RESOURCE) {
    	STRINGorID xml(_ttoi(GetSkinFile().c_str()));
    	auto callback = nbase::Bind(&WindowImplBase::CreateControl, this, std::placeholders::_1);
        pRoot = (Box*)builder.Create(xml, callback, this);
    }
    else {
        auto callback = nbase::Bind(&WindowImplBase::CreateControl, this, std::placeholders::_1);
        pRoot = (Box*)builder.Create((GetWindowResourcePath() + GetSkinFile()).c_str(), callback, this);
    }
    ASSERT(pRoot);
    if (pRoot == NULL) {
    	MessageBox(NULL, _T("加载资源文件失败"), _T("Duilib"), MB_OK|MB_ICONERROR);
    	return -1;
    }

    // ...
}
```

**我们可以看到，两者本质上都是通过`WindowBuilder`完成解析操作**
# WindowBuilder
`WindowBuilder`在`CMarkup`的基础上，完成duilib特化的工作
```c++
class Box;
class Window;
typedef std::function<Control* (const std::wstring&)> CreateControlCallback;

class UILIB_API WindowBuilder
{
public:
    WindowBuilder();
    
    Box* Create(STRINGorID xml, CreateControlCallback pCallback = CreateControlCallback(),
		Window* pManager = nullptr, Box* pParent = nullptr, Box* pUserDefinedBox = nullptr);
    Box* Create(CreateControlCallback pCallback = CreateControlCallback(), Window* pManager = nullptr,
		Box* pParent = nullptr, Box* pUserDefinedBox = nullptr);

    CMarkup* GetMarkup();

    void GetLastErrorMessage(LPTSTR pstrMessage, SIZE_T cchMax) const;
    void GetLastErrorLocation(LPTSTR pstrSource, SIZE_T cchMax) const;

private:
    Control* _Parse(CMarkupNode* parent, Control* pParent = NULL, Window* pManager = NULL);
    Control* CreateControlByClass(const std::wstring& strControlClass);
    void AttachXmlEvent(bool bBubbled, CMarkupNode& node, Control* pParent);

private:
    CMarkup m_xml;
    CreateControlCallback m_createControlCallback;
};
```
`CreateControlCallback`是自定义控件的回调函数，根据标签值返回对应控件类

`Create`函数主要生成布局

`_Parse`函数主要生成控件

# Global标签、Window标签解析
这两个最外层的标签在`Create`函数被处理

`Create`函数先处理两者的属性
```c++
if (strClass == _T("Global"))
{
	int nAttributes = root.GetAttributeCount();
	for (int i = 0; i < nAttributes; i++)
	{
		// global 支持的属性
		strName = root.GetAttributeName(i);
		strValue = root.GetAttributeValue(i);
		if (strName == _T("disabledfontcolor"))
		{
			GlobalManager::SetDefaultDisabledTextColor(strValue);
		}
		else if (strName == _T("defaultfontcolor"))
		{
			GlobalManager::SetDefaultTextColor(strValue);
		}
		else if (strName == _T("linkfontcolor"))
		{
			DWORD clrColor = GlobalManager::GetTextColor(strValue);
			GlobalManager::SetDefaultLinkFontColor(clrColor);
		}
		else if (strName == _T("linkhoverfontcolor"))
		{
			DWORD clrColor = GlobalManager::GetTextColor(strValue);
			GlobalManager::SetDefaultLinkHoverFontColor(clrColor);
		}
		else if (strName == _T("selectedcolor"))
		{
			DWORD clrColor = GlobalManager::GetTextColor(strValue);
			GlobalManager::SetDefaultSelectedBkColor(clrColor);
		}
	}
}
else if (strClass == _T("Window"))
{
	if (pManager->GetHWND())
	{
		int nAttributes = root.GetAttributeCount();
		for (int i = 0; i < nAttributes; i++)
		{
			// window 支持的属性
			strName = root.GetAttributeName(i);
			strValue = root.GetAttributeValue(i);
			if (strName == _T("size"))
			{
				LPTSTR pstr = NULL;
				int cx = _tcstol(strValue.c_str(), &pstr, 10);	ASSERT(pstr);
				int cy = _tcstol(pstr + 1, &pstr, 10);	ASSERT(pstr);
				pManager->SetInitSize(cx, cy);
			}
			else if (strName == _T("heightpercent"))
			{
				double lfHeightPercent = _ttof(strValue.c_str());
				pManager->SetHeightPercent(lfHeightPercent);

				MONITORINFO oMonitor = {};
				oMonitor.cbSize = sizeof(oMonitor);
				::GetMonitorInfo(::MonitorFromWindow(pManager->GetHWND(), MONITOR_DEFAULTTOPRIMARY), &oMonitor);
				int nWindowHeight = int((oMonitor.rcWork.bottom - oMonitor.rcWork.top) * lfHeightPercent);
				int nMinHeight = pManager->GetMinInfo().cy;
				int nMaxHeight = pManager->GetMaxInfo().cy;
				if (nMinHeight != 0 && nWindowHeight < nMinHeight)
				{
					nWindowHeight = nMinHeight;
				}
				if (nMaxHeight != 0 && nWindowHeight > nMaxHeight)
				{
					nWindowHeight = nMaxHeight;
				}

				CSize xy = pManager->GetInitSize();
				pManager->SetInitSize(xy.cx, nWindowHeight, false, false);
			}
			else if (strName == _T("sizebox"))
			{
				UiRect rcSizeBox;
				LPTSTR pstr = NULL;
				rcSizeBox.left = _tcstol(strValue.c_str(), &pstr, 10);  ASSERT(pstr);
				rcSizeBox.top = _tcstol(pstr + 1, &pstr, 10);    ASSERT(pstr);
				rcSizeBox.right = _tcstol(pstr + 1, &pstr, 10);  ASSERT(pstr);
				rcSizeBox.bottom = _tcstol(pstr + 1, &pstr, 10); ASSERT(pstr);
				pManager->SetSizeBox(rcSizeBox);
			}
			else if (strName == _T("caption"))
			{
				UiRect rcCaption;
				LPTSTR pstr = NULL;
				rcCaption.left = _tcstol(strValue.c_str(), &pstr, 10);  ASSERT(pstr);
				rcCaption.top = _tcstol(pstr + 1, &pstr, 10);    ASSERT(pstr);
				rcCaption.right = _tcstol(pstr + 1, &pstr, 10);  ASSERT(pstr);
				rcCaption.bottom = _tcstol(pstr + 1, &pstr, 10); ASSERT(pstr);
				pManager->SetCaptionRect(rcCaption);
			}
			else if (strName == _T("textid"))
			{
				pManager->SetTextId(strValue);
			}
			else if (strName == _T("roundcorner"))
			{
				LPTSTR pstr = NULL;
				int cx = _tcstol(strValue.c_str(), &pstr, 10);  ASSERT(pstr);
				int cy = _tcstol(pstr + 1, &pstr, 10);    ASSERT(pstr);
				pManager->SetRoundCorner(cx, cy);
			}
			else if (strName == _T("mininfo"))
			{
				LPTSTR pstr = NULL;
				int cx = _tcstol(strValue.c_str(), &pstr, 10);  ASSERT(pstr);
				int cy = _tcstol(pstr + 1, &pstr, 10);    ASSERT(pstr);
				pManager->SetMinInfo(cx, cy);
			}
			else if (strName == _T("maxinfo"))
			{
				LPTSTR pstr = NULL;
				int cx = _tcstol(strValue.c_str(), &pstr, 10);  ASSERT(pstr);
				int cy = _tcstol(pstr + 1, &pstr, 10);    ASSERT(pstr);
				pManager->SetMaxInfo(cx, cy);
			}
			else if (strName == _T("shadowattached"))
			{
				pManager->SetShadowAttached(strValue == _T("true"));
			}
			else if (strName == _T("shadowimage"))
			{
				pManager->SetShadowImage(strValue);
			}
			else if (strName == _T("shadowcorner"))
			{
				UiRect rc;
				LPTSTR pstr = NULL;
				rc.left = _tcstol(strValue.c_str(), &pstr, 10);  ASSERT(pstr);
				rc.top = _tcstol(pstr + 1, &pstr, 10);    ASSERT(pstr);
				rc.right = _tcstol(pstr + 1, &pstr, 10);  ASSERT(pstr);
				rc.bottom = _tcstol(pstr + 1, &pstr, 10); ASSERT(pstr);
				pManager->SetShadowCorner(rc);
			}
			else if (strName == _T("alphafixcorner") || strName == _T("custom_shadow"))
			{
				UiRect rc;
				LPTSTR pstr = NULL;
				rc.left = _tcstol(strValue.c_str(), &pstr, 10);  ASSERT(pstr);
				rc.top = _tcstol(pstr + 1, &pstr, 10);    ASSERT(pstr);
				rc.right = _tcstol(pstr + 1, &pstr, 10);  ASSERT(pstr);
				rc.bottom = _tcstol(pstr + 1, &pstr, 10); ASSERT(pstr);
				pManager->SetAlphaFixCorner(rc);
			}
		}
	}
}
```
`<Global>`标签支持的属性有
- `disabledfontcolor`
- `defaultfontcolor`
- `linkfontcolor`
- `linkhoverfontcolor`
- `selectedcolor`

`Window`标签支持的属性有
- `size` 窗口大小
- `heightpercent`
- `sizebox`
- `caption`	允许拖动的rect
- `textid` 窗口标题文字
- `roundcorner` 圆角
- `mininfo`
- `maxinfo`
- `shadowattached` 阴影是否算点击
- `shadowimage` 阴影图
- `shadowcorner` 阴影圆角
- `alphafixcorner`

`Create`函数再处理两者的子标签
```c++
if (strClass == _T("Global"))
{
	for (CMarkupNode node = root.GetChild(); node.IsValid(); node = node.GetSibling())
	{
		strClass = node.GetName();
		if (strClass == _T("Image"))
		{
			ASSERT(FALSE);	//废弃
		}
		else if (strClass == _T("FontResource"))
		{
			nAttributes = node.GetAttributeCount();
			std::wstring strFontFile;
			std::wstring strFontName;
			for (int i = 0; i < nAttributes; i++)
			{
				strName = node.GetAttributeName(i);
				strValue = node.GetAttributeValue(i);
				if (strName == _T("file"))
				{
					strFontFile = strValue;
				}
				else if (strName == _T("name"))
				{
					strFontName = strValue;
				}
			}
			if (!strFontFile.empty())
			{
				FontManager::GetInstance()->AddFontResource(strFontFile, strFontName);
			}
		}
		else if (strClass == _T("Font"))
		{
			nAttributes = node.GetAttributeCount();
			std::wstring strFontName;
			int size = 12;
			bool bold = false;
			bool underline = false;
			bool italic = false;
			for (int i = 0; i < nAttributes; i++)
			{
				strName = node.GetAttributeName(i);
				strValue = node.GetAttributeValue(i);
				if (strName == _T("name"))
				{
					strFontName = strValue;
				}
				else if (strName == _T("size"))
				{
					size = _tcstol(strValue.c_str(), &pstr, 10);
				}
				else if (strName == _T("bold"))
				{
					bold = (strValue == _T("true"));
				}
				else if (strName == _T("underline"))
				{
					underline = (strValue == _T("true"));
				}
				else if (strName == _T("italic"))
				{
					italic = (strValue == _T("true"));
				}
				else if (strName == _T("default"))
				{
					ASSERT(FALSE);//废弃
				}
			}
			if (!strFontName.empty())
			{
				GlobalManager::AddFont(strFontName, size, bold, underline, italic);
			}
		}
		else if (strClass == _T("Class"))
		{
			nAttributes = node.GetAttributeCount();
			std::wstring strClassName;
			std::wstring strAttribute;
			for (int i = 0; i < nAttributes; i++)
			{
				strName = node.GetAttributeName(i);
				strValue = node.GetAttributeValue(i);
				if (strName == _T("name"))
				{
					strClassName = strValue;
				}
				else if (strName == _T("value"))
				{
					strAttribute = strValue;
				}
			}
			if (!strClassName.empty())
			{
				GlobalManager::AddClass(strClassName, strAttribute);
			}
		}
		else if (strClass == _T("TextColor"))
		{
			nAttributes = node.GetAttributeCount();
			std::wstring strColorName;
			std::wstring strColor;
			for (int i = 0; i < nAttributes; i++)
			{
				strName = node.GetAttributeName(i);
				strValue = node.GetAttributeValue(i);
				if (strName == _T("name"))
				{
					strColorName = strValue;
				}
				else if (strName == _T("value"))
				{
					strColor = strValue;
				}
			}
			if (!strColorName.empty())
			{
				GlobalManager::AddTextColor(strColorName, strColor);
			}
		}
	}
}
else if (strClass == _T("Window"))
{
	for (CMarkupNode node = root.GetChild(); node.IsValid(); node = node.GetSibling())
	{
		strClass = node.GetName();
		if (strClass == _T("Class"))
		{
			nAttributes = node.GetAttributeCount();
			std::wstring strClassName;
			std::wstring strAttribute;
			for (int i = 0; i < nAttributes; i++)
			{
				strName = node.GetAttributeName(i);
				strValue = node.GetAttributeValue(i);
				if (strName == _T("name"))
				{
					strClassName = strValue;
				}
				else if (strName == _T("value"))
				{
					strAttribute = strValue;
				}
			}
			if (!strClassName.empty())
			{
				ASSERT(GlobalManager::GetClassAttributes(strClassName).empty());	//窗口中的Class不能与全局的重名
				pManager->AddClass(strClassName, strAttribute);
			}
		}
	}
}
```

`<Global>`标签支持的子标签有
- `FontResource`
- `Font` 字体，它支持的属性有
	- `name`
	- `size`
	- `bold`
	- `underline`
	- `italic`
- `Class` 样式类
- `TextColor`

`Window`标签在这层只支持
- `Class` 样式类

完成了`<Global>`标签与`<Window>`标签的解析后，`Create`对xml子节点递归进行其他解析
```c++
for( CMarkupNode node = root.GetChild() ; node.IsValid(); node = node.GetSibling() ) {
	std::wstring strClass = node.GetName();
	if (strClass == _T("Image") || strClass == _T("FontResource") || strClass == _T("Font")
		|| strClass == _T("Class") || strClass == _T("TextColor") ) {
		// 这些都在上面处理了，忽略
	}
	else {
		if (!pUserDefinedBox) {
			return (Box*)_Parse(&root, pParent, pManager);
		}
		else {
			int nAttributes = node.GetAttributeCount();
			for( int i = 0; i < nAttributes; i++ ) {
				// 对于所有控件来说，class必须是第一个属性
				ASSERT(i == 0 || _tcscmp(node.GetAttributeName(i), _T("class")) != 0);
				pUserDefinedBox->SetAttribute(node.GetAttributeName(i), node.GetAttributeValue(i));
			}
			
			_Parse(&node, pUserDefinedBox, pManager);
			return pUserDefinedBox;
		}
	}
}
```
# Include标签、控件标签、事件标签解析
主要通过`_Parse`进行其他解析
```c++
Control* WindowBuilder::_Parse(CMarkupNode* pRoot, Control* pParent, Window* pManager)
{
    Control* pReturn = NULL;
    for( CMarkupNode node = pRoot->GetChild() ; node.IsValid(); node = node.GetSibling() ) {
		std::wstring strClass = node.GetName();
		// 上面处理过了
		if( strClass == _T("Image") || strClass == _T("Font")
			|| strClass == _T("Class") || strClass == _T("TextColor") ) {
				continue;
		}

        Control* pControl = NULL;
		// Include标签
        if( strClass == _T("Include") ) {
            if( !node.HasAttributes() ) continue;
            int nCount = 1;
            LPTSTR pstr = NULL;
            TCHAR szValue[500] = { 0 };
            SIZE_T cchLen = lengthof(szValue) - 1;
            if ( node.GetAttributeValue(_T("count"), szValue, cchLen) )
                nCount = _tcstol(szValue, &pstr, 10);
            cchLen = lengthof(szValue) - 1;
            if ( !node.GetAttributeValue(_T("source"), szValue, cchLen) ) continue;
            for ( int i = 0; i < nCount; i++ ) {
		// 递归解析
                WindowBuilder builder;
                pControl = builder.Create((LPCTSTR)szValue, m_createControlCallback, pManager, (Box*)pParent);
            }
            continue;
        }
        else {
			// 标准duilib控件
			pControl = CreateControlByClass(strClass);
			if (pControl == nullptr) {
				if (strClass == L"Event" || strClass == L"BubbledEvent") {
					bool bBubbled = (strClass == L"BubbledEvent");
					AttachXmlEvent(bBubbled, node, pParent);
					continue;
				}
			}

            // 自定义控件
            if( pControl == NULL ) {
				pControl = GlobalManager::CreateControl(strClass);
            }

            if( pControl == NULL && m_createControlCallback ) {
                pControl = m_createControlCallback(strClass);
            }
        }

		if( pControl == NULL ) {
			ASSERT(FALSE);
			continue;
		}

		pControl->SetWindow(pManager);
		// Process attributes
		if( node.HasAttributes() ) {
			// Set ordinary attributes
			int nAttributes = node.GetAttributeCount();
			for( int i = 0; i < nAttributes; i++ ) {
				ASSERT(i == 0 || _tcscmp(node.GetAttributeName(i), _T("class")) != 0);	//class必须是第一个属性
				pControl->SetAttribute(node.GetAttributeName(i), node.GetAttributeValue(i));
			}
		}

        // Add children
        if( node.HasChildren() ) {
            _Parse(&node, (Box*)pControl, pManager);
        }

		// Attach to parent
        // 因为某些属性和父窗口相关，比如selected，必须先Add到父窗口
		if( pParent != NULL ) {
			Box* pContainer = dynamic_cast<Box*>(pParent);
			ASSERT(pContainer);
			if( pContainer == NULL ) return NULL;
			if( !pContainer->Add(pControl) ) {
				ASSERT(FALSE);
				delete pControl;
				continue;
			}
		}
        
        // Return first item
        if( pReturn == NULL ) pReturn = pControl;
    }
    return pReturn;
}
```
- **Include**
	`Include`标签实际上是将目标文件拷贝到此处，所以在这里做递归处理
- **duilib标准控件**
	交给`CreateControlByClass`处理
	```c++
	Control* WindowBuilder::CreateControlByClass(const std::wstring& strControlClass)
	{
		Control* pControl = nullptr;
		SIZE_T cchLen = strControlClass.length();
		switch( cchLen ) {
		case 3:
			if( strControlClass == DUI_CTR_BOX )					pControl = new Box;
			break;
		case 4:
			if( strControlClass == DUI_CTR_HBOX )					pControl = new HBox;
			else if( strControlClass == DUI_CTR_VBOX )				pControl = new VBox;
			break;
		case 5:
			if( strControlClass == DUI_CTR_COMBO )                  pControl = new Combo;
			else if( strControlClass == DUI_CTR_LABEL )             pControl = new Label;
			break;
		case 6:
			if( strControlClass == DUI_CTR_BUTTON )                 pControl = new Button;
			else if( strControlClass == DUI_CTR_OPTION )            pControl = new Option;
			else if( strControlClass == DUI_CTR_SLIDER )            pControl = new Slider;
			else if( strControlClass == DUI_CTR_TABBOX )			pControl = new TabBox;
			break;
		case 7:
			if( strControlClass == DUI_CTR_CONTROL )                pControl = new Control;
			else if( strControlClass == DUI_CTR_TILEBOX )		  	pControl = new TileBox;
			else if (strControlClass == DUI_CTR_LISTBOX)			pControl = new ListBox(new Layout);
			//else if( pstrClass == DUI_CTR_ACTIVEX )				pControl = new ActiveX;
			break;
		case 8:
			if( strControlClass == DUI_CTR_PROGRESS )               pControl = new Progress;
			else if( strControlClass == DUI_CTR_RICHEDIT )          pControl = new RichEdit;
			else if( strControlClass == DUI_CTR_CHECKBOX )			pControl = new CheckBox;
			//else if( pstrClass == DUI_CTR_DATETIME )				pControl = new DateTime;
			else if( strControlClass == DUI_CTR_TREEVIEW )			pControl = new TreeView;
			else if( strControlClass == DUI_CTR_TREENODE )			pControl = new TreeNode;
			else if( strControlClass == DUI_CTR_HLISTBOX )			pControl = new ListBox(new HLayout);
			else if( strControlClass == DUI_CTR_VLISTBOX )          pControl = new ListBox(new VLayout);
			else if ( strControlClass == DUI_CTR_CHILDBOX )			pControl = new ChildBox;
			else if( strControlClass == DUI_CTR_LABELBOX )          pControl = new LabelBox;
			break;
		case 9:
			if( strControlClass == DUI_CTR_SCROLLBAR )				pControl = new ScrollBar; 
			else if( strControlClass == DUI_CTR_BUTTONBOX )         pControl = new ButtonBox;
			else if( strControlClass == DUI_CTR_OPTIONBOX )         pControl = new OptionBox;
			break;
		case 10:
			//if( pstrClass == DUI_CTR_WEBBROWSER )					pControl = new WebBrowser;
			break;
		case 11:
			if( strControlClass == DUI_CTR_TILELISTBOX )			pControl = new ListBox(new TileLayout);
			else if( strControlClass == DUI_CTR_CHECKBOXBOX )		pControl = new CheckBoxBox;
			break;
		case 14:
			if (strControlClass == DUI_CTR_VIRTUALLISTBOX)			pControl = new VirtualListBox;
			break;
		case 15:
			break;
		case 16:
			break;
		case 20:
			if( strControlClass == DUI_CTR_LISTCONTAINERELEMENT )   pControl = new ListContainerElement;
			break;
		}

		return pControl;
	}
	```
- **自定义控件**
	由`Create`传入的`CreateControlCallback`回调决定
- **XML事件**
	包括两个标签`<Event>`与`<BubbleEvent>`