# 库


## url切块
```python
from urllib import parse
url = 'http://www.baidu.com?name=a#flag'
print(parse.urlparse)
# scheme='http', netloc='www.baidu.com', path='', params='', query='name=a', fragment='flag'
```

## 避免导包执行
```python
func() # import 之后会调用

if __name__ == '__main__':
    func()  # 从此文件中执行才会调用
```

## 解析json的两种方法
- request自带的简单json
```python
html = requests.get(url)
data = html.json()['data']    # 取出json中的data字段
```
- 官方json包
```python
import json

html = requests.get(url)
data = json.loads(html.text)['data']
```

## 切块下载（图片或者视频）
```python
import uuid # 生成随机数
def downloadimg(url):
    img = requests.get(url)
    with open('imgs/{}.jpg'.format(uuid.uuid4()), 'wb') as f:
        chunks = img.iter_content(125)  # 125字节分片，减少cpu压力
        for c in chunks:
            f.write(c)
```

## 使用session
- 自动保存cookie
- 自动记录参数如User-Agent
```python
session = requests.session()
session.headers=headers
# 之后的requests可以替换成session
html = session.get(url)
```

## 忽略HTTPS证书验证
```python
html = requests.get(url, verify=False)
```

## 代理IP
网址：[快代理](https://www.kuaidaili.com/free/inha/)，[西刺代理](https://www.xicidaili.com/)
```python
proxies={
    'http': 'http://123.123.123.123:12345',
    'https': 'https://123.123.123.123:12345'
}
html = session.get(url, proxies=proxies
                    timeout=3)  # 防止IP失效超时
```

## requests_html包
- 不需要使用解析包
- 可以执行js
- 异步

## 网页编码与pycharm编码不一致
例如网页编码使用gbk，而pycharm的默认编码是utf-8，这样在ide上面会显示乱码
```python
html = requests.get('www.baidu.com')
html.encoding='utf-8'
```

## 字符串格式化方法
- C语言风格
```python
res = 'a = %s, %d' % ('a', 1)
```
- format
```python
res = 'a = {}'.format('a')
```

- f 3.7新特性
```python
insert_str = 'a'
res = f'a = {insert_str}'
```

## 正则表达式
- `re.match`从头开始搜索，`re.search`全局搜索
- 返回的对象是一个类，`group`函数返回全部，`groups`函数返回括号内
```python
target = 'I am a boy'
res = re.search('a (.*?)', target, re.I)    # boy
```
- `compile`函数将正则解耦出来
- `sub`函数可以正则替换

## XPath
```python
from lxml import etree
target = '''
<li class="li li-first">
    <a href="link.html">
        first item
    </a>
</li>
'''
html = etree.HTML(target)
```
可以在F12中的html文档中Ctrl+F搜索栏下使用XPath语法

## with语法糖
实际上调用了面向对象中的`__enter__`以及`__close__`方法

## csv文件
逗号分隔值(Comma-Separated Values, CSV)，文件以纯文本形式存储表格数据
```
Name, Age, Sex
Mike, 15, Boy
Lily, 15, Gril
```
python使用
```python
import csv

with open('./test.csv', encoding='urf-8') as f: # 配合'a+'封装成方法
    f_csv = csv.writer(f)
    f_csv.writerow(('Name', 'Age', 'Sex'))
    f_csv.writerow([('Mike', 15, 'Boy'), ('Lily', '15', 'Gril')])

with open('./test.csv', encoding='utf-8') as f:
    f_csv=csv.reader(f) # 列表 DictReader字典
    next(f_csv)
    for c in f_csv:
        print(c)

# result 
# ['Mike', '15', 'Boy']
# ['Lily', '15', 'Gril']
```

## Excel
```python
import xlwt # 写入
import xlrd # 读取

def create():
    excel_book = xlwt.Workbook()
    sheet = excel_book.add_sheet("sheet1")
    sheet.write(1, 0, "hello")
    excel_book.save('test.xls')

def read():
    excel_book = xlrd.open_workbook('text.xls')
    sheet = excel_book.sheets()[0]
    row_size = excel_book.sheets()[0].nrows
    col_size = excel_book.sheets()[0].ncols
    for i in range(row_size):
        print(sheet.row_values(i))
    for j in range(col_size):
        print(sheet.col_values(j))
    v_1_1 = sheet.cell_value(1,1)
```

## word
pip install python-docx
```python
form docx import Document
# useless
```

## SQL建表
```python
from sqlalchemy import create_engine, Table, MetaData, Column, String, Integer, select
from sqlalchemy import ForeignKey
from sqlalchemy.orm import sessionmaker, relationship
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()   # 建表需要继承的基类
class TableA(Base):
    __tablename__ = 'TableA'
    Name = Column(Integer(), primary_key = True)    # int主键
    Value = Column(String(10), nullable = False)      # 10字符非空
    TableB_Id = Column(Integer(), ForeignKey("TableB.Id"))  # 外键关联TableB的Id项

class TableB(Base):
    __tablename__ = 'TableB'
    Id = Column(Integer(), primary_key = True, autoincrement = True)    # 自增主键
```