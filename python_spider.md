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