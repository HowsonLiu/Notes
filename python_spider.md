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
ORM
> Object-Relational Mapping 将数据库映射成对象
```python
from sqlalchemy import create_engine, Table, MetaData, Column, String, Integer, select
from sqlalchemy import ForeignKey
from sqlalchemy.orm import sessionmaker, relationship
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()   # 建表需要继承的基类
class TableA(Base):
    __tablename__ = 'TableA'    # 数据库中表的名字
    Name = Column(Integer(), primary_key = True)    # int主键
    Value = Column(String(10), nullable = False)      # 10字符非空
    TableB_Id = Column(Integer(), ForeignKey('TableB.Id'))  # 外键关联TableB的Id项

class TableB(Base):
    __tablename__ = 'TableB'
    Id = Column(Integer(), primary_key = True, autoincrement = True)    # 自增主键
```

## Sqlalchemy relationship
```python
class Father(Base):
    __tablename__ = 'Father'
    name = Column(String(125), primary_key = True)
    son = relationship('Son', backref = 'father'        # 1
        cascade = 'all, delete' )       # 删除时自动清除关联数据

class Son(Base):
    __tablename__ = 'Son'
    name = Column(String(125), primary_key = True)
    father_name = Column(String(100), ForeignKey('Father.name'))    # 2
```
在这里，Father跟Son是一对多的关系。在Son表中，包含了外键`father_name`，来源是Father表的`name`字段（2）。在Father表中，有一个`relationship`（1），表示Father表可以通过`.son`来关联他的Son表，同时`backref`字段表示Son表可以通过`.father`来关联他的Father表，用于简化操作。

## Sqlalchemy Add
```python
engine = create_engine(
    'mysql+pymysql://root:password@127.0.0.1:3306/database_name'
    max_overflow = 5,   # 最大连接数
    pool_size = 10      # 线程数量
    echo = True         # 回显
)
session = sessionmaker(bind = engine)
sess = session()
f1 = Father(name = 'FatherOne')
sess.add(f1)
sess.commit()   # 提交事务才能生效

s1 = Son(name = 'SonOne')
s2 = Son(name = 'SonTwo')
s1.father = f1
s2.father = f2      # 使用了上面的relationship的backref
sess.add_all([s1, s2])
sess.commit()

f2 = Father(name = 'FatherTwo')
f2.son = [Son(name = 'SonThree'), Son(name = 'SonFour')]    # relationship
sess.add(f2)
sess.commit()
```

## Sqlalchemy Search and Delete and update
```python
res = sess.query(Son).filter_by(name = 'SonOne')  # select * from Son where name = 'SonOne'
sess.delete(res)
from sqlalchemy import or_, and_
sess.query(Son).filter(or_(Son.name == 'SonOne', Son.name == 'SonTwo')).all()   # select * from Son where name = 'SonOne' or name = 'SonTwo'
# Son.name.like('Son%')     模糊查询
s2.name = 'Son2'    # 这样就更新了
sess.commit()
```

## 多线程连接数据库
```python
from sqlalchemy.orm import scoped_session
Session = sessionmaker(engines)
session = scoped_session(Session)   # 保证线程安全

class MyThread(threading.Thread):
    def __init__(self, threadName):
        super(MyThread, self).__init__()
        self.name = threadName

    def run(self):
        # 每个线程一个session
        sess = session()

if __name__ == '__main__':
    thread_pool = []
    for i in range(10):
        thread_pool.append(MyThread('%s' % i))
    for i in thread_pool:
        i.start()
    for i in thread_pool:
        i.join()
```
同一线程同一session，不同线程不同session

## Python中的TLS
```python
import threading
a = threading.local()
a.value = 1

def change(val):
    try:
        print(threading.current_thread().name, a.value)
    except:
        print(threading.current_thread().name, 'no a.value')    # 三个线程都打印这个

for i in range(3):
    threading.Thread(target = change, args = (i)).start()

print(threading.current_thread().name, a.value)     # 打印出1
```
a变量虽然是全局的，但是不同线程会有一份属于自己的拷贝

## google fire包
允许在命令行中传入参数
```python
import fire

def func(val):
    print(val)

if __name__ == '__main__':
    fire.Fire(func)
```
> python 1.py --val=123   # 123

## python GIL
> GIL: 全局解释器锁(global interpreter lock)。即使在多核心处理的机器上，GIL解析器保证任何时刻仅有一个线程在执行
- 多线程在计算密集型时，不好用，因为GIL，建议用C
- 在IO密集型时，好用，因为网络，磁盘IO阻塞情况比较多，用C替换提升不大，反而用Python代码量少
### thread的解决方案
使用多进程`multiprocessing`代替多线程`thread`，缺点是：
- 开销大
- 进程通信难

## python thread and lock
```python
import threading

lock = threading.Lock()
rlock = thread.RLock()  # 递归锁
# with rlock:
#   do sth

def func(arg):
    lock.acquire()
    print(arg)
    lock.release()

t = threading.Thread(target = func, args = ('hello',))   # 仅声明
t.setDaemon(True)   # 守护线程，随着主线程消失 
t.start()   # 执行
t.join()    # 主线程阻塞等待t执行完
```

## python multiprocessing
```python
import multiprocessing

p = multiprocessing.Process(target = func, args = ('hello',))
p.start()
p.join()
```
### 进程通信
共享内存
```python
from multiprocessing import Process, Queue
def write(q):
    for i in range(10):
        q.put(i)

def read(q):
    while True:
        i = q.get()

q = Queue()
pw = Process(target = write, args = (q,))
pr = Process(target = read, args = (q,))
pw.start()
pr.start()
pw.join()
```

## python pool
```python
def square(val):
    return val * val

inputs = list(range(100))   # [0,1,2,3,...]
pool = multiprocessing.Pool(processes = 4)
outputs =pool.map(square, inputs)   # [0,1,4,9,...]
output = pool.apply(square, args = (1,))    # 单任务
pool.close()
pool.join()
```
>pip install threadpool
```python
import threadpool
pool = threadpool.ThreadPool(10)    # 线程数
requests = threadpool.makeRequests(square, inputs)   # 提交任务
for req in requests:
    pool.putRequest(req)    # 执行任务
pool.wait()     # join?
```

## python concurrent
3.2版本内置支持
```python
import concurrent.futures

with concurrent.futures.ThreadPoolExecutor(max_workers = 5) as executor:
    futures = [executor.submit(func, val) for val in vals]
    for future in concurrent.futures.as_completed(futures):
        print(future.result())
```

## 协程
协程修饰函数，用来对有大量IO操作的程序进行加速
- 异步函数前加上`async`关键字
- 调用异步函数不能直接调用，需要加上`await`
- 与异步函数交互的函数也是异步函数
```python
import asyncio

async def get_html(url):
    asyncio.sleep(3)    # 注意不是time.sleep

tasks = []
for url in urls:
    tasks.append(get_html(url))
loop = asyncio.get_event_loop()
loop.run_until_complete(asymcio.wait(tasks))
# 单任务
task = asyncio.create_task(get_html())
await task
```
requests库不支持异步爬虫，使用aiohttp

## aiohttp
```python
import asyncio
import aiohttp

async def get_html(url):
    async with aio.http.ClientSession(headers = headers) as session:
        async with session.get(url) as resp:
            text = await resp.text()

loop = asyncio.get_event_loop()
tasks = [get_html(url) for url in urls]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

## 分布式爬虫
Docker容器，可以理解为轻量级的虚拟机