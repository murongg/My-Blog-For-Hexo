---
title: 爬取某站所有性感美女图片（Python版）
date: 2020-01-18 19:20:20
thumbnail: https://s2.ax1x.com/2020/01/19/1CS6HO.png
categories:
  - 程序世界
  - python
tags:
  - 爬虫
---

## Python 版

### 环境

- python3.7
- pip19.3

### 依赖

- requests
- pyquery
- os

### 前期准备

#### 引入所需依赖

```python
import requests
import os
from requests.packages import urllib3
from pyquery import PyQuery as pq
```
<!--more-->
### headers 设置（重要）

```js
headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) '
                  'AppleWebKit/537.36 (KHTML, like Gecko)'
                  'Chrome/58.0.3029.110 Safari/537.36'
}
```

### 目标站

http://www.umei.cc/p/gaoqing/cn/
![目标站截图](https://s2.ax1x.com/2020/01/19/1CS2Ue.png)
![](https://s2.ax1x.com/2020/01/19/1CS6HO.png)

### 获取需要爬取页面的地址

```python
def get_detail(url):
    """
    获取需要爬取页面的地址
    :param url:
    :return:
    """
    urllib3.disable_warnings()
    html = requests.get(url, headers=headers, verify=False)
    html.encoding = 'utf-8'
    code = html.status_code
    if code == 404:
        print('本页面URL已爬取结束')
        return code
    doc = pq(html.text)
    a = doc('.TypeBigPics')
    print(a)
    end_list = []
    for item in a.items():
        url_result = item.attr('href')
        result = 'htm' in url_result.split('/')[-1]
        if result:
            end_list.append(url_result.split('.htm')[0])
    return end_list
```

### 获取每一页所有图片

```python
def get_img(url):
    """
    获取每一页所有图片
    :param url:
    :return:
    """
    urllib3.disable_warnings()
    html = requests.get(url, headers=headers, verify=False)
    html.encoding = 'utf-8'
    code = html.status_code
    if code == 404:
        print('本页面图片已爬取结束')
        return True
    doc = pq(html.text)
    img = doc('.wrap .ImageBody a img')
    img_title = img.attr('alt')
    if not img_title:
        return True
    img_url = img.attr('src')
    root = "图片文件夹路径（需提前建好）//" + img_title + '/'
    path = root + img_url.split('/')[-1]
    # 根目录加上url中以反斜杠分割的最后一部分，即可以以图片原来的名字存储在本地
    try:
        if not os.path.exists(root):  # 判断当前根目录是否存在
            print('创建根目录')
            os.mkdir(root)  # 创建根目录
        if not os.path.exists(path):  # 判断文件是否存在
            r = requests.get(img_url)
            with open(path, 'wb')as f:
                f.write(r.content)
                f.close()
                print("文件保存成功", '\n', '\n')
        else:
            print("文件已存在")
    except:
        print("爬取失败")
```

### main 主函数

```python
if __name__ == '__main__':
    z = 1
    n = 1
    url = 'http://www.umei.cc/p/gaoqing/cn/'
    for d in range(n, 26):
        # 提前看好了 一共有26页，所以提前设置好循环次数
        pa_url = '{0}{1}.htm'.format(url, d)
        print(pa_url)
        result = get_detail(pa_url)
        for e in result:
            for i in range(z, 100):
                if i == 1:
                    url1 = e + '.htm'
                else:
                    url1 = e + '_' + str(i) + '.htm'
                is_end = get_img(url1)
                # 如果状态码为404，则进行下次循环
                if is_end == True:
                    break
```

## 成果

![文件夹](https://s2.ax1x.com/2020/01/19/1CSoKP.png)
![文件个数](https://s2.ax1x.com/2020/01/19/1CSXCj.png)
