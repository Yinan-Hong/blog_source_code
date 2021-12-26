+++
author = "兔楠"
title = "微博评论区爬虫"
date = "2021-12-26"
description = "testing..."
tags = [
    "爬虫","微博"
]
categories = [
    "小工具"
]
toc = true

+++



因为要做python期末大作业，所以写了个爬虫爬微博的评论区。

这几天正好有王力宏的瓜，于是就爬了一下王力宏的评论区，分析一下高频词汇什么的，分析大众心理哈哈哈...

<!--more-->

## 太长不看版

[<font style="color:#6495ed">SHOW ME THE CODE</font>](https://yinanhong.site/post/wlhspider/#完整代码)


## 微博的三个客户端

微博有三个客户端：

```
weibo.com // 网页端
m.weibo.cn // 手机端
weibo.cn // 移动端
```

网页端前端页面比较复杂，好像还会加密数据，比较难操作。相对的来说，**手机端和移动端**比较好爬。



## 移动端爬虫

微博移动端就是这样的2G网页面，前端页面非常简单，html代码中的评论部分都会有清晰的标记，可以很方便地用正则表达式提取出评论内容。

```python
comments = re.findall('<span class="ctt">(.*?)</span>', html)
```



<img src="../wlhSpider/Snipaste_2021-12-26_16-00-14.png" alt="Snipaste_2021-12-26_16-00-14" style="zoom:38%;" />

移动端的**分页机制**也很简单，通过该域名里的`&page=`就可以访问不同的页面。

```python
# 爬取一页评论内容
def get_one_page(url):
    headers = {
        'User-agent' : 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36 Edg/96.0.1054.62',
        'Host' : 'weibo.cn',
        'Accept' : 'application/json, text/plain, */*',
        'Accept-Language' : 'zh-CN,zh;q=0.9',
        'Accept-Encoding' : 'gzip, deflate, br',
        'Cookie' : '换成你的cookie',
        'DNT' : '1',
        'Connection' : 'keep-alive'
    }
    # 获取网页 html
    response = requests.get(url, headers = headers, verify=False)
    # 爬取成功
    if response.status_code == 200:
        # 返回值为 html 文档，传入到解析函数当中
        return response.text
    return None

# 访问所有页数
for i in range(1, max_page):
    url = 'https://weibo.cn/comment/L6w2sfDXb?uid=5977512966&rl=0&page='+str(i)
    html = get_one_page(url)
    print('正在爬取第 %d 页评论' % (i+1))
    save_one_page(html)
    time.sleep(1 + random.random()) # 防止被反爬
```

好像这样很简单，但是移动端**50页之后就访问不到数据**了。应该不是反爬策略，因为手动翻到 >50 的页面也是没有显示评论内容的。于是放弃移动端，改爬手机端。



## 手机端爬虫

搜了一圈发现，以前手机端的分页机制是和移动端一样的，通过修改域名结尾的`&page=`就可以访问不同页数的评论，爬起来比较容易。但是好像在2019年微博有一次改版，变更了分页机制。

### 手机端评论的分页机制

现在微博评论是往下滑之后会加载出下一页评论，而这是如何向服务器发请求的呢？

<img src="../wlhSpider/85ee1f3ba90366d55a76245bd022c218 (1).gif" alt="85ee1f3ba90366d55a76245bd022c218 (1)" style="zoom: 50%;" />

用浏览器打开`m.weibo.cn`，登录之后访问到对应页面。按`F12`打开浏览器开发者工具，然后往下划，加载评论的时候观察网络请求。

![Snipaste_2021-12-26_17-45-47](../wlhSpider/Snipaste_2021-12-26_17-45-47.png)

其实手机端就是通过这个请求获取下一页评论的。

我们再观察一下请求域名，每一页的请求不同的地方是`max_id`。

![Snipaste_2021-12-26_17-48-06](../wlhSpider/Snipaste_2021-12-26_17-48-06.png)

看一下每次请求传回来的json数据，里面就包含了下一页请求的`max_id`。还有就是，`max`是最大页数，我爬的这条王力宏的微博有七十多万条评论，最大页数是三万多。

![Snipaste_2021-12-26_17-48-53](../wlhSpider/Snipaste_2021-12-26_17-48-53.png)

再看下第一页的请求，发现是没有`max_id`的。那就好办了，只需要访问第一页，将每次收到的`max_id`填入下一次的请求里就行了。关键代码如下。

```python
# 爬取第一页的微博评论
def first_page_comment(weibo_id, url, headers):
    global commentLists
    try:
        url = url + str(weibo_id) + '&mid=' + str(weibo_id) + '&max_id_type=0' # 得到请求域名
        web_data = requests.get(url, headers=headers, cookies=Cookie, timeout=20)
        js_con = web_data.json() # 解析出json包

        max_id = js_con['data']['max_id'] # 下一页的max_id
        max = js_con['data']['max'] # 获取最大页数
        comments_list = js_con['data']['data'] # 提取评论内容
        extract_data(comments_list)

        print("已获取第1页的评论")
        return max_id, max, commentLists

# 爬取剩下的页数
def get_rest_comments(weibo_id, url, headers, max, max_id):
    global commentLists
    urlNew = ''
    count = 2 # 记录当前页数
    while count <= max:
        try:
            urlNew = url + str(weibo_id) + '&mid=' + str(weibo_id) + \
                '&max_id=' + str(max_id) + '&max_id_type=0' # 得到下一次请求域名

            web_data = requests.get(url=urlNew, headers=headers, cookies=Cookie, timeout=10)
            if web_data.status_code == 200:
                js_con = web_data.json()

                if js_con['ok'] == 1:
                    max_id = js_con['data']['max_id'] # 下一次的max_id
                    comments_list = js_con['data']['data'] # 提取评论内容
                    extract_data(comments_list)
                    print("已获取第" + str(count) + "页的评论。")
                    count += 1

max_id, max_page, output = first_page_comment(item['weibo_id'], url, headers)
get_rest_comments(item['weibo_id'], url, headers, max_page, max_id)
```



---------------------------------

### 这部分内容有误，可以略过...

按照上面的代码能顺利地爬到第十五页。问题就出在，从第十六页的请求开始，域名结尾的`&max_id_type=0`会变成`&max_id_type=1`。

![Snipaste_2021-12-26_17-54-04](../wlhSpider/Snipaste_2021-12-26_17-54-04.png)

比较蛋疼的是，从十六页往后，绝大多数情况是`&max_id_type=1`，但偶尔还是会有`&max_id_type=0`。如果`&max_id_type=`填错了，是会有数据返回的，而且是这样的。

<img src="../wlhSpider/Snipaste_2021-12-26_17-52-23.png" alt="Snipaste_2021-12-26_17-52-23" style="zoom: 50%;" />

那么只需要做一个小的调整。因为绝大多数情况是`=1`的，如果请求返回`'ok'=0`就改成`&max_id_type=0`再发一次。

```python
# 出现max_id_type=0的情况
    if js_con['ok'] == 0:
       if urlNew[-1] == '1':
            urlNew = urlNew[:-1] + '0'
        else:
            urlNew = urlNew[:-1] + '1'
        continue
```

**（后来发现下一次请求的max_id_type也会在上一次返回的数据里给到**

---------------------------

手机端上每一页的评论有十多条的样子，比移动端多多了。

![Snipaste_2021-12-26_18-58-32](../wlhSpider/Snipaste_2021-12-26_18-58-32.png)



### 保存评论数据和请求参数

因为偶尔遇到网络错误，所以要时常保存一下已经读好的数据。请求参数也要保存，因为`max_id`是看不出规律的，我要爬的这条微博有三万多页评论，如果程序挂了要从头爬会很费时间的。

我这里是每500页写入一下评论数据，然后记录一下请求参数。

除此之外，还可以做一个异常处理，发生异常时把当前已经读取到的数据写入磁盘，然后记录一下当前的请求参数。这部分的实现就不贴出来了

```python
def write_in(index: str):
    global commentLists
    txt_name = 'res/wlh_' + str(index) + '.txt'
    with open(txt_name, mode='w', encoding='utf-8') as file_handle:
        for obj in commentLists:
            file_handle.write(obj['comment_text'] + '\n')

# 每500页写入一下txt
if count % 500 == 0:
    # 格式化txt文件名称：wlh_{起始页}-{终止页}.txt
    index = str(last) + '-' + str(count)
    last = count + 1
    write_in(index)
    # 清空前面的数据
    commentLists = []
    # 记录一下当前的url，以免出错后要从头开始爬
    with open('log.txt', 'a') as log:
        msg = '\nMark\n' + urlNew + '\ncnt = ' + str(count) + '\n'
        log.write(msg)
```



## 数据分析

就是用jieba分词，然后再进行频数统计，过滤掉禁用词之类的，很常规的操作了。

```python
wordcount = {}
for file in comment_files:
    content = open('wlh_res/' + file, 'rb').read()
    print('read ', file)
    # jieba 分词
    word_list = jieba.cut(content)
    words = []
    # 过滤掉stopwords
    for word in word_list:
        if word not in stop_words:
            words.append(word)
	# 统计频数
    for word in words:
        if word != ' ':
            wordcount[word] = wordcount.get(word, 0)+1
# 排个序
wordtop = sorted(wordcount.items(), key=lambda x: x[1], reverse=True)[1:25]
```

然后使用了pyecharts这个可视化工具，生成出好看的图表。就不展开介绍这个工具和用法了。

运行后会生成一个`render.html`就是图表，可以直接在浏览器里打开。



## 完整代码

 [<font style="color:#6495ed">Yinan-Hong/weibo_spider: 微博评论爬虫 (github.com)</font>](https://github.com/Yinan-Hong/weibo_spider)

现在自己的设备上登录微博，访问到对应的界面。按`F12`然后选择Fetch/XHR，如果没有显示cookie的话是因为浏览器有缓存，需要按`ctrl`+`F5`刷新一下。

![Snipaste_2021-12-26_18-11-57](../wlhSpider/Snipaste_2021-12-26_18-11-57.png)

找到对应的信息，填到config.py里。`weibo_id`就是域名里的那串数字，代表对应的博文id。

<img src="../wlhSpider/Snipaste_2021-12-26_18-14-25.png" alt="Snipaste_2021-12-26_18-14-25" style="zoom: 50%;" />

填好之后运行wlh_spider.py就行了。运行正常的话，会打印出`已获取第n页的评论。`

如果中途程序异常终止，运行`reboot.py`即可。

## Acknowledgement

爬虫模块参考了 [<font style="color:#6495ed">https://blog.csdn.net/qq_40511157/article/details/104216546</font>](https://blog.csdn.net/qq_40511157/article/details/104216546)

分析模块参考了 [<font style="color:#6495ed">王力宏的瓜很大！我用Python爬取了瓜文评论区，发现更精彩 (qq.com)</font>](https://mp.weixin.qq.com/s/4Vb4l2ze4hnrErSVduVkGA)

