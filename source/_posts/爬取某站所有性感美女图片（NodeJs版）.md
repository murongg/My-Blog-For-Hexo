---
title: 爬取某站所有性感美女图片（NodeJs版）
date: 2020-01-20 11:16:13
thumbnail: https://s2.ax1x.com/2020/01/20/1PYTRe.png
categories:
  - 程序世界
  - NodeJs
tags:
  - 爬虫
---
## 准备工作
### 环境
- node v10.13.0
- npm v6.4.1
### 项目依赖
- request-promise（网络请求promise版）
- cheerio（dom操作）
- fs（文件读写）
### 其他
#### 路径
```javascript
const BASE_URL = "http://www.umei.cc/p/gaoqing/cn/"; // 目标站地址
const BASE_PATH = "D://picture//"; // 需要存储到的文件夹
```
#### headers设置
```javascript
const headers = {
  "User-Agent":
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36"
};
```
<!--more-->
## 开始
### 首页分析
![目标站首页](https://s2.ax1x.com/2020/01/20/1P3439.png)

#### 编写页码遍历代码
在首页我们需要获取每一篇图集的完整URL地址，和首页列表的页码。

![底部页码截图](https://s2.ax1x.com/2020/01/20/1P8uD0.png)

点击末页可以发现一共有26页，写死就好了，现在可以编写第一步的代码了。
``` javascript
/**
 * 一共有26页，提前做好遍历，并拼接URL
 *
 */
async function getPageList() {
  for (let index = 1; index <= 26; index++) {
    // 最终URL示例http://www.umei.cc/p/gaoqing/cn/26.htm
    const url = `${BASE_URL}${index}.htm`;
    // 下一步所需操作方法
    await getPageUrl(url);
  }
}
```

接下来就要获取每个图集的详情URL地址

![首页dom](https://s2.ax1x.com/2020/01/20/1PGpRJ.png)

根据dom可以发现详情的a标签地址在`.TypeList ul li a.TypeBigPics`的`href`里  
#### 编写获取图集详情地址代码
``` javascript
/**
 * 获取每一页的列表URL
 *
 * @param {*} baseUrl
 */
async function getPageUrl(baseUrl) {
  const body = await request({
    url: baseUrl
  });
  const $ = cheerio.load(body);
  const urls = $(".TypeList ul li a.TypeBigPics");
  urls.each(async function() {
    const url = $(this).attr("href");
    console.log(url);
    if (url) {
      // 下一操作，获取每篇图集的所有页码链接
      await getDetailUrl(url);
    }
  });
}
```
### 详情页分析
现在我们就可以进入图集详情页了，在图集详情页依然要获取详细的页码信息，以此获取到图集的所有图片，并且下载到本地。
#### 获取页码
![图集详情页页码](https://s2.ax1x.com/2020/01/20/1PJSTf.png)

这次我们不能将页码写死了，要取出dom`.NewPages ul li a`里的`共`与`页`之间的数字，这里要注意，取出来的值是`String`类型，我们要把他转为`Number`类型，取到页码后，还需要将每个图集的`title`取出来，用来生成需要保存图片到的文件夹。
```javascript
/**
 * 获取每篇图集的所有链接
 *
 * @param {*} url
 * @returns
 */
async function getDetailUrl(url) {
  const body = await request({
    url: url
  });
  const $ = cheerio.load(body);
  // 正则获取有多少页
  const count = Number(
    $(".NewPages ul li a")
      .first()
      .text()
      .match(/共(\S*)页/)[1]
  );
  // 生成要保存的图片文件夹
  const path =
    BASE_PATH +
    $(".ArticleTitle")
      .text()
      .trim();
  const isMkdir = mkdirFolder(path);
  if (!isMkdir) {
    return;
  }
  let defaultUrl = url.split(".htm")[0];
  // 遍历页码，生成图集详情页每一页的URL
  for (let index = 1; index <= count; index++) {
    let endUrl = "";
    if (index === 1) {
      endUrl = defaultUrl + ".htm";
    } else {
      endUrl = `${defaultUrl}_${index}.htm`;
    }
    // 下一操作，获取每一页图片的URL地址
    await getDetail(endUrl, path);
  }
}
```
#### 创建文件夹方法封装
```javascript
/**
 * 创建文件夹
 *
 * @param {*} path
 * @returns
 */
function mkdirFolder(path) {
  if (!fs.existsSync(path)) {
    //查看是否存在这个文件夹
    fs.mkdirSync(path); //不存在就建文件夹
    console.log(`${path}文件夹创建成功`);
    return true;
  } else {
    console.log(`${path}文件夹已经存在`);
    return true;
  }
}
```
#### 获取图集详情每页图片的URL

![img标签的dom](https://s2.ax1x.com/2020/01/20/1PYV8e.png)

由图可见，我们需要获取`.ImageBody p a img`的`src`属性。

```javascript
/**
 * 获取每一页的图片URL
 *
 * @param {*} detailUrl
 * @returns
 */
async function getDetail(detailUrl, path) {
  const body = await request({
    url: detailUrl,
    headers
  });
  const $ = cheerio.load(body);

  const imgUrl = $(".ImageBody p a img").attr("src");
  if (imgUrl) {
    const imgTitle = imgUrl.split("/").pop();
    // 下一操作，保存图片到本地
    await getImage(imgUrl, imgTitle, path);
  }
}
```
#### 保存图片方法

利用`fs`库，进行文件操作
```javascript
/**
 * 获取每一页所有图片
 *
 * @param {*} imgUrl
 * @param {*} imgTitle
 * @param {*} path
 */
async function getImage(imgUrl, imgTitle, path) {
  const imgRes = await request({
    url: imgUrl,
    headers,
    resolveWithFullResponse: true
  }).pipe(fs.createWriteStream(`${path}/${imgTitle}`));
  if (imgRes) {
    console.log(`[${imgTitle}]保存成功`);
  }
}

```
### 运行

```shell
node meinv.js
```

![运行过程](https://s2.ax1x.com/2020/01/20/1PY6xJ.png)

不得不说，js的异步太恶心了，图片一张没下载下来，文件夹全给我建完了。

再看看带宽占用。  

![带宽占用](https://s2.ax1x.com/2020/01/20/1PY4IK.png)

![](https://s2.ax1x.com/2020/01/20/1PYTRe.png)

## 全部代码
``` javascript
const request = require("request-promise"); // 网络请求
const cheerio = require("cheerio"); // 操作dom
const fs = require("fs"); // 读写文件
const BASE_URL = "http://www.umei.cc/p/gaoqing/cn/"; // 目标站地址
const BASE_PATH = "D://picture//"; // 需要存储到的文件夹

const headers = {
  "User-Agent":
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36"
};

/**
 * 一共有26页，提前做好遍历，并拼接URL
 *
 */
async function getPageList() {
  for (let index = 1; index <= 26; index++) {
    // 最终URL示例http://www.umei.cc/p/gaoqing/cn/26.htm
    const url = `${BASE_URL}${index}.htm`;
    // 下一步所需操作方法
    await getPageUrl(url);
  }
}
/**
 * 获取每一页的列表URL
 *
 * @param {*} baseUrl
 */
async function getPageUrl(baseUrl) {
  const body = await request({
    url: baseUrl
  });
  const $ = cheerio.load(body);
  const urls = $(".TypeList ul li a.TypeBigPics");
  urls.each(async function() {
    const url = $(this).attr("href");
    if (url) {
      // 下一操作，获取每篇图集的所有页码链接
      await getDetailUrl(url);
    }
  });
}

/**
 * 获取每篇图集的所有链接
 *
 * @param {*} url
 * @returns
 */
async function getDetailUrl(url) {
  const body = await request({
    url: url
  });
  const $ = cheerio.load(body);
  // 正则获取有多少页
  const count = Number(
    $(".NewPages ul li a")
      .first()
      .text()
      .match(/共(\S*)页/)[1]
  );
  // 生成要保存的图片文件夹
  const path =
    BASE_PATH +
    $(".ArticleTitle")
      .text()
      .trim();
  const isMkdir = mkdirFolder(path);
  if (!isMkdir) {
    return;
  }
  let defaultUrl = url.split(".htm")[0];
  // 遍历页码，生成图集详情页每一页的URL
  for (let index = 1; index <= count; index++) {
    let endUrl = "";
    if (index === 1) {
      endUrl = defaultUrl + ".htm";
    } else {
      endUrl = `${defaultUrl}_${index}.htm`;
    }
    // 下一操作，获取每一页图片的URL地址
    await getDetail(endUrl, path);
  }
}
/**
 * 获取每一页的图片URL
 *
 * @param {*} detailUrl
 * @returns
 */
async function getDetail(detailUrl, path) {
  const body = await request({
    url: detailUrl,
    headers
  });
  const $ = cheerio.load(body);

  const imgUrl = $(".ImageBody p a img").attr("src");
  if (imgUrl) {
    const imgTitle = imgUrl.split("/").pop();
    // 下一操作，保存图片到本地
    await getImage(imgUrl, imgTitle, path);
  }
}

/**
 * 获取每一页所有图片
 *
 * @param {*} imgUrl
 * @param {*} imgTitle
 * @param {*} path
 */
async function getImage(imgUrl, imgTitle, path) {
  const imgRes = await request({
    url: imgUrl,
    headers,
    resolveWithFullResponse: true
  }).pipe(fs.createWriteStream(`${path}/${imgTitle}`));
  if (imgRes) {
    console.log(`[${imgTitle}]保存成功`);
  }
}

/**
 * 创建文件夹
 *
 * @param {*} path
 * @returns
 */
function mkdirFolder(path) {
  if (!fs.existsSync(path)) {
    //查看是否存在这个文件夹
    fs.mkdirSync(path); //不存在就建文件夹
    console.log(`${path}文件夹创建成功`);
    return true;
  } else {
    console.log(`${path}文件夹已经存在`);
    return true;
  }
}

getPageList();
```