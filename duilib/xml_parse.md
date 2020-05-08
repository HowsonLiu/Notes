# 网易duilib中的xml解析（一）CMarkup
# 数据结构
xml树由`CMarkup`表达
```c++
class UILIB_API CMarkup
{
public:
    CMarkup(LPCTSTR pstrXML = NULL);
    ~CMarkup();

    // ...
private:
    LPTSTR m_pstrXML;                   // xml原字符串
    XMLELEMENT* m_pElements;            // xml节点数组
    ULONG m_nElements;                  // 当前xml节点数量
    ULONG m_nReservedElements;          // 容器最大xml节点数量
    TCHAR m_szErrorMsg[100];            // 错误信息
    TCHAR m_szErrorXML[50];             // 错误所在位置
    bool m_bPreserveWhitespace;         // 是否保留空格
}
```
xml节点在整个xml树中的关系由`XMLELEMENT`表达
```c++
typedef struct tagXMLELEMENT
{
    ULONG iStart;       // 节点起始下标
    ULONG iChild;       // 首个子节点起始下标
    ULONG iNext;        // 下一个兄弟节点起始下标
    ULONG iParent;      // 父节点起始下标
    ULONG iData;        // 节点内数据项起始下标
} XMLELEMENT;
```
xml节点的完整信息由`CMarkupNode`表达
```c++
class UILIB_API CMarkupNode
{
    friend class CMarkup;
private:
    CMarkupNode();
    CMarkupNode(CMarkup* pOwner, int iPos);

public:
    bool IsValid() const;

    // xml位置信息
    CMarkupNode GetParent();
    CMarkupNode GetSibling();
    CMarkupNode GetChild();
    CMarkupNode GetChild(LPCTSTR pstrName);
    bool HasSiblings() const;
    bool HasChildren() const;

    // xml属性信息
    LPCTSTR GetName() const;
    LPCTSTR GetValue() const;
    bool HasAttributes();
    bool HasAttribute(LPCTSTR pstrName);
    int GetAttributeCount();
    LPCTSTR GetAttributeName(int iIndex);
    LPCTSTR GetAttributeValue(int iIndex);
    LPCTSTR GetAttributeValue(LPCTSTR pstrName);
    bool GetAttributeValue(int iIndex, LPTSTR pstrValue, SIZE_T cchMax);
    bool GetAttributeValue(LPCTSTR pstrName, LPTSTR pstrValue, SIZE_T cchMax);

    // ...
};
```
我们可以看到
- 虽然xml在语法上面有树形结构，但duilib没有使用树形的数据结构表示，而是使用顺序结构表示（`XMLELEMENT* m_pElements`）
- xml节点有两种表达：轻量级的`tagXMLELEMENT`仅仅提供位置信息，给`CMarkup`用于快速遍历；重量级的`CMarkupNode`提供全部信息，给外部使用
- `CMarkup`与`CMarkupNode`都不会开辟额外的内存空间拷贝节点信息。结构的描述都以索引的形式去描述，索引对应的地址是一个由`CMarkup`维护的原xml字符串
# 无关语义忽略
`CMarkup`解析时只检查树格式是否正确，因此它使用以下函数跳过其他信息
```c++
void CMarkup::_SkipWhitespace(LPCTSTR& pstr) const
{
    while( *pstr > _T('\0') && *pstr <= _T(' ') ) pstr = ::CharNext(pstr);
}

void CMarkup::_SkipWhitespace(LPTSTR& pstr) const
{
    while( *pstr > _T('\0') && *pstr <= _T(' ') ) pstr = ::CharNext(pstr);
}

void CMarkup::_SkipIdentifier(LPCTSTR& pstr) const
{
    // 属性只能用英文，所以这样处理没有问题
    while( *pstr != _T('\0') && (*pstr == _T('_') || *pstr == _T(':') || _istalnum(*pstr)) ) pstr = ::CharNext(pstr);
}

void CMarkup::_SkipIdentifier(LPTSTR& pstr) const
{
    // 属性只能用英文，所以这样处理没有问题
    while( *pstr != _T('\0') && (*pstr == _T('_') || *pstr == _T(':') || _istalnum(*pstr)) ) pstr = ::CharNext(pstr);
}
```
`_SkipWhitespace`跳过所有不可视的字符

`_SkipIdentifier`跳过节点类型，包括命名空间，例如
```
<f:table>
   <f:name>African Coffee Table</f:name>
   <f:width>80</f:width>
   <f:length>120</f:length>
</f:table>
```
但在duilib上面不存在这种写法。这种写法能绕过这层，但在上层**WindowBuilder**会报错
# xml树形解析
```c++
bool CMarkup::_Parse(LPTSTR& pstrText, ULONG iParent)
{
    _SkipWhitespace(pstrText);
    ULONG iPrevious = 0;
    for( ; ; ) 
    {
        // 空串仅在第一层有效，否则属于结束异常
        if( *pstrText == _T('\0') && iParent <= 1 ) return true;
        _SkipWhitespace(pstrText);

        // 检查起始 <
        if( *pstrText != _T('<') ) return _Failed(_T("Expected start tag"), pstrText);
        if( pstrText[1] == _T('/') ) return true;
        *pstrText++ = _T('\0');
        _SkipWhitespace(pstrText);

        // 跳过注释
        // <?xml version="1.0" encoding="utf-8"?>
        // <!--comoent-->
        if( *pstrText == _T('!') || *pstrText == _T('?') ) {
            TCHAR ch = *pstrText;
            if( *pstrText == _T('!') ) ch = _T('-');
            while( *pstrText != _T('\0') && !(*pstrText == ch && *(pstrText + 1) == _T('>')) ) pstrText = ::CharNext(pstrText);
            if( *pstrText != _T('\0') ) pstrText += 2;
            _SkipWhitespace(pstrText);
            continue;
        }
        _SkipWhitespace(pstrText);

        // 记录节点
        XMLELEMENT* pEl = _ReserveElement();
        ULONG iPos = pEl - m_pElements;
        pEl->iStart = pstrText - m_pstrXML;     // 起始<的下标
        pEl->iParent = iParent;                 // 父节点的下标
        pEl->iNext = pEl->iChild = 0;           // 子节点、弟弟节点
        if( iPrevious != 0 ) m_pElements[iPrevious].iNext = iPos;       // 哥哥节点填写此节点为弟弟节点
        else if( iParent > 0 ) m_pElements[iParent].iChild = iPos;      // 父节点填写此节点为子节点
        iPrevious = iPos;
        // 暂时记录节点类型字符串，给后面判断结尾使用
        LPCTSTR pstrName = pstrText;        // 类型字符串首
        _SkipIdentifier(pstrText);
        LPTSTR pstrNameEnd = pstrText;      // 类型字符串尾
        if( *pstrText == _T('\0') ) return _Failed(_T("Error parsing element name"), pstrText);
        // 检查节点属性
        if( !_ParseAttributes(pstrText) ) return false;
        _SkipWhitespace(pstrText);
        if( pstrText[0] == _T('/') && pstrText[1] == _T('>') )      // 单个标签  即<Control/>
        {
            pEl->iData = pstrText - m_pstrXML;
            *pstrText = _T('\0');
            pstrText += 2;
        }
        else    // 一对标签 即<Control></Control>
        {
            if( *pstrText != _T('>') ) return _Failed(_T("Expected start-tag closing"), pstrText);
            // 记录属性起始下标
            pEl->iData = ++pstrText - m_pstrXML;
            LPTSTR pstrDest = pstrText;
            // 解析数据
            // <Box>123</Box>中的123
            if( !_ParseData(pstrText, pstrDest, _T('<')) ) return false;
            // 检查标签头之后的数据
            if( *pstrText == _T('\0') && iParent <= 1 ) return true;
            if( *pstrText != _T('<') ) return _Failed(_T("Expected end-tag start"), pstrText);
            if( pstrText[0] == _T('<') && pstrText[1] != _T('/') ) 
            {
                if( !_Parse(pstrText, iPos) ) return false;     // 遍历子节点
            }
            // 检查标签对属性是否匹配 如<Box></Control>
            if( pstrText[0] == _T('<') && pstrText[1] == _T('/') ) 
            {
                *pstrDest = _T('\0');
                *pstrText = _T('\0');
                pstrText += 2;
                _SkipWhitespace(pstrText);
                SIZE_T cchName = pstrNameEnd - pstrName;
                if( _tcsncmp(pstrText, pstrName, cchName) != 0 ) return _Failed(_T("Unmatched closing tag"), pstrText);
                pstrText += cchName;
                _SkipWhitespace(pstrText);
                if( *pstrText++ != _T('>') ) return _Failed(_T("Unmatched closing tag"), pstrText);
            }
        }
        *pstrNameEnd = _T('\0');
        _SkipWhitespace(pstrText);
    }
}
```
解析xml树形用的是`_Parse`，使用**深度优先**的**递归**遍历

它接受两个输入：`pstrText`xml字符串，`iParent`父节点索引

它的输出是`XMLELEMENT`数组`m_pElements`，数组从上而下依次对应xml字符串从上而下遍历得到的xml节点
# xml转义字符解析
```c++
void CMarkup::_ParseMetaChar(LPTSTR& pstrText, LPTSTR& pstrDest)
{
    if( pstrText[0] == _T('a') && pstrText[1] == _T('m') && pstrText[2] == _T('p') && pstrText[3] == _T(';') ) {
        *pstrDest++ = _T('&');
        pstrText += 4;
    }
    else if( pstrText[0] == _T('l') && pstrText[1] == _T('t') && pstrText[2] == _T(';') ) {
        *pstrDest++ = _T('<');
        pstrText += 3;
    }
    else if( pstrText[0] == _T('g') && pstrText[1] == _T('t') && pstrText[2] == _T(';') ) {
        *pstrDest++ = _T('>');
        pstrText += 3;
    }
    else if( pstrText[0] == _T('q') && pstrText[1] == _T('u') && pstrText[2] == _T('o') && pstrText[3] == _T('t') && pstrText[4] == _T(';') ) {
        *pstrDest++ = _T('\"');
        pstrText += 5;
    }
    else if( pstrText[0] == _T('a') && pstrText[1] == _T('p') && pstrText[2] == _T('o') && pstrText[3] == _T('s') && pstrText[4] == _T(';') ) {
        *pstrDest++ = _T('\'');
        pstrText += 5;
    }
    else {
        *pstrDest++ = _T('&');
    }
}
```
`pstrText`作为输入，`pstrDest`作为转义后的输出

支持的转义表如下
|原字符串|转义后字符串|
:---:|:---:
`&amp;`|&
`&lt;`|<
`&gt;`|>
`&quot;`|"
`&apos;`|'
# xml数据解析
```c++
bool CMarkup::_ParseData(LPTSTR& pstrText, LPTSTR& pstrDest, char cEnd)
{
    while( *pstrText != _T('\0') && *pstrText != cEnd ) {
        // 解析转义字符
        if( *pstrText == _T('&') ) {
            while( *pstrText == _T('&') ) {
            	_ParseMetaChar(++pstrText, pstrDest);
            }
            if (*pstrText == cEnd)
            	break;
        }
        // 压缩空格
        if( *pstrText == _T(' ') ) {
            *pstrDest++ = *pstrText++;
            if( !m_bPreserveWhitespace ) _SkipWhitespace(pstrText);
        }
        else {
            LPTSTR pstrTemp = ::CharNext(pstrText);
            while( pstrText < pstrTemp) {
                *pstrDest++ = *pstrText++;
            }
        }
    }
    // Make sure that MapAttributes() works correctly when it parses
    // over a value that has been transformed.
    LPTSTR pstrFill = pstrDest + 1;
    while( pstrFill < pstrText ) *pstrFill++ = _T(' ');
    return true;
}
```
`_ParseData`主要用来转义字符串以及压缩空格

`pstrText`是原字符串，`pstrDest`是转义输出位置，`cEnd`是停止字符串。`cEnd`在本例中取两个值：" 以及 < ，依次处理**属性值转义**`<Class name="font_title" value="font=&quot;2&quot;" />`和**数据值转义**`<Box>&quot</Box>`两种情况（*第二种情况duilib不做处理*）
# xml属性解析
```c++
bool CMarkup::_ParseAttributes(LPTSTR& pstrText)
{   
    if( *pstrText == _T('>') ) return true;     // 空属性
    *pstrText++ = _T('\0');
    _SkipWhitespace(pstrText);
    while( *pstrText != _T('\0') && *pstrText != _T('>') && *pstrText != _T('/') ) {
        _SkipIdentifier(pstrText);
        LPTSTR pstrIdentifierEnd = pstrText;
        _SkipWhitespace(pstrText);
        if( *pstrText != _T('=') ) return _Failed(_T("Error while parsing attributes"), pstrText);
        *pstrText++ = _T(' ');
        *pstrIdentifierEnd = _T('\0');
        _SkipWhitespace(pstrText);
        if( *pstrText++ != _T('\"') ) return _Failed(_T("Expected attribute value"), pstrText);
        LPTSTR pstrDest = pstrText;
        if( !_ParseData(pstrText, pstrDest, _T('\"')) ) return false;
        if( *pstrText == _T('\0') ) return _Failed(_T("Error while parsing attribute string"), pstrText);
        *pstrDest = _T('\0');
        if( pstrText != pstrDest ) *pstrText = _T(' ');
        pstrText++;
        _SkipWhitespace(pstrText);
    }
    return true;
}
```
`CMarkup::_ParseAttributes`用来检查属性格式是否正确以及转义属性值，真正要取属性键值对需要借助`CMarkupNode`的相关函数

`CMarkupNode`提取键值对的函数如下
```c++
class UILIB_API CMarkupNode
{
private:
    CMarkupNode();
    CMarkupNode(CMarkup* pOwner, int iPos);
    // ...
private:
    void _MapAttributes();

    enum { MAX_XML_ATTRIBUTES = 64 };
    // 属性对表示
    typedef struct
    {
        ULONG iName;
        ULONG iValue;
    } XMLATTRIBUTE;

    int m_iPos;
    int m_nAttributes;
    XMLATTRIBUTE m_aAttributes[MAX_XML_ATTRIBUTES];
    CMarkup* m_pOwner;
};

void CMarkupNode::_MapAttributes()
{
    m_nAttributes = 0;
    LPCTSTR pstr = m_pOwner->m_pstrXML + m_pOwner->m_pElements[m_iPos].iStart;
    LPCTSTR pstrEnd = m_pOwner->m_pstrXML + m_pOwner->m_pElements[m_iPos].iData;
    pstr += _tcslen(pstr) + 1;
    while( pstr < pstrEnd ) {
        m_pOwner->_SkipWhitespace(pstr);
        // 将键值对依次放入属性对数组中
        m_aAttributes[m_nAttributes].iName = pstr - m_pOwner->m_pstrXML;
        pstr += _tcslen(pstr) + 1;
        m_pOwner->_SkipWhitespace(pstr);
        if( *pstr++ != _T('\"') ) return; // if( *pstr != _T('\"') ) { pstr = ::CharNext(pstr); return; }
        
        m_aAttributes[m_nAttributes++].iValue = pstr - m_pOwner->m_pstrXML;
        if( m_nAttributes >= MAX_XML_ATTRIBUTES ) return;
        pstr += _tcslen(pstr) + 1;
    }
}
```
`CMarkupNode`最高支持**64**个属性，超过会忽略不算

内部使用`_MapAttributes`函数，从`CMarkup`中取**首节点**字符串，解析后，将键值对依次存到`XMLATTRIBUTE`数组中
# 总结
在这一层里主要是做标准xml格式的检查，duilib特化的工作主要在**WindowBuilder**进行