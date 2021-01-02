---
title: Python爬虫-m3u8视频爬取
mathjax: true
date: 2021-01-02 10:48:05
excerpt: 使用python爬取流媒体网站的m3u8视频
categories: 爬虫
tags: ['爬虫','Python']
keywords: ['m3u8','爬虫','python' ]

---



m3u8文件+ts文件是很多流媒体网站常用的一种方法，本文作为爬虫练习项目，记录了如何使用python爬虫爬取某视频网站的视频资源。

# 一、分析

第一步是确定想要爬取的资源地址，通过网页源代码找到资源的url。

F12进入开发者模式，找到m3u8后缀的文件，可以看到有两个，把第一个m3u8文件下载下来以后发现，其内容是第二个m3u8的地址，第二个m3u8的url才是真实的地址

![](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/crawler-wuchizhitu1.png)

![第一个m3u8文件内容](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/crawler-wuchizhitu3.png)



可见，第一个m3u8文件的内容，是真正的m3u8的地址。



![](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/crawler-wuchizhitu2.png)



![第二个m3u8文件内容](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/crawler-wuchizhitu4.png)



第二个m3u8文件中的内容才是真正的ts文件的地址。

每一集视频是由多个ts文件构成的，将这些ts文件拼接起来就是完整的一集内容。这些ts文件的url都保存到第二个m3u8文件中。因此可以确定大体流程：

1. 获取当前集的m3u8地址，并下载m3u8文件。
2. 从m3u8文件中获取ts视频的url。
3. 根据ts文件的url下载视频。
4. 将多个ts文件合并，得到完整的一集内容，保存到相应路径。



#  二、获取m3u8文件

分析网页源代码，可以看到m3u8的url保存在一个playurls的列表中，并且一季的所有集的地址都在，因此只需要在其中一集的网页源代码中获取出当前季的所有集的url即可：

```python
# 获取每季中每集的m3u8地址,每季只获取一次即可
def get_m3u8_list(url,S):
    req = requests.get(url)
    req.encoding = 'utf-8'
    html = req.text
    # 使用正则表达式从网页代码中找到m3u8的地址
    res_url = re.findall(r'https:\\/\\/youku.com-youku.net.*?index.m3u8', html, re.S)
    m3u8list = []
    for i in range(len(res_url)):
        url = res_url[i].split('\\')
        # m3u8文件下载下来以后，文件内容才是真正的m3u8地址，这里偷个懒，手动构建url
        # 真正的url多了'1000k/hls',手动添加上即可
        m3u8list.append(''.join(url[:-1])+'/1000k/hls/index.m3u8')
        print(m3u8list[i])
    print('第{}季m3u8地址获取完毕'.format(S))
    return m3u8list
```

常规思路是获取到第一层m3u8文件，然后从中获取到真正的m3u8的地址，通过分析，真正的url是在第一层的url后面两个文件路径(仅对于当前视频网站，具体需要根据实际情况分析)，这里手动添加上了，没有读取文件。



# 三、获取ts文件url并下载

获取到m3u8文件的地址后，就可以下载文件，通过读取文件内容，获取到当前集的所有ts文件url，然后下载。

```python
# 下载文件
def download(m3u8_list,base_path,S):  # base_path: "F://Shameless//",S表示当前季数
    print('下载m3u8文件...')
    url = base_path+'Shameless_'+'S'+str(S)  # F://Shameless//Shameless_S1
    path = Path(url)
    # 如果文件夹不存在，则创建
    if not path.is_dir():
        os.mkdir(url)
    for i in range(len(m3u8_list)):
        print('正在下载第{}集...'.format(i+1))
        start = datetime.datetime.now().replace(microsecond=0)
        time.sleep(1)  # sleep一秒
        ts_urls = []  # 保存每一集的ts文件的真实url
        m3u8 = requests.get(url=m3u8_list[i])
        content = m3u8.text.split('\n')
        # 获取ts文件地址
        for s in content: 
            if s.endswith('.ts'): 
                ts_url = m3u8_list[i][:-10] + s.strip('\n') # 生成ts文件的真实url
                ts_urls.append(ts_url)
        download_ts(ts_urls,down_path=url+'//'+"E"+str(i+1)+'.ts')  # 根据ts的url下载每集的ts文件
        end = datetime.datetime.now().replace(microsecond=0)
        print('耗时：%s' % (end - start))
        print('第{}集下载完成...'.format(i+1))


# 根据ts下载链接下载文件，并合并为完整的视频文件
def download_ts(ts_urls,down_path):
    file = open(down_path, 'wb')  # 这里将每个ts文件添加到file里面，即合并
    for i in tqdm(range(len(ts_urls))):
        ts_url = ts_urls[i]  # 例:https://youku.com-youku.net/20180626/14084_f3588039/1000k/hls/80ed70a101f861.ts
        time.sleep(1)
        try:
            response = requests.get(url=ts_url, stream=True, verify=False)
            file.write(response.content)
        except Exception as e:
            print('异常请求：%s' % e.args)
    file.close()
```

# 四、总结

## 完整代码

完整代码如下：

```python
import requests
import re
import os
from pathlib import Path
import time
import datetime
from tqdm import tqdm

import urllib3
urllib3.disable_warnings()  # 禁用证书认证和警告


# 获取每季中每集的m3u8地址,每季只获取一次即可
def get_m3u8_list(url,S):
    req = requests.get(url)
    req.encoding = 'utf-8'
    html = req.text
    res_url = re.findall(r'https:\\/\\/youku.com-youku.net.*?index.m3u8', html, re.S)
    m3u8list = []
    for i in range(len(res_url)):
        url = res_url[i].split('\\')
        # m3u8文件下载下来以后，文件内容才是真正的m3u8地址，这里为了方便起见，手动构建url
        # 真正的url多了'1000k/hls',这里手动添加上
        m3u8list.append(''.join(url[:-1])+'/1000k/hls/index.m3u8')
        print(m3u8list[i])
    print('第{}季m3u8地址获取完毕'.format(S))
    return m3u8list


# 下载ts文件
def download(m3u8_list,base_path,S):  # base_path: "F://Shameless//",S表示当前季数
    print('下载m3u8文件...')
    url = base_path+'Shameless_'+'S'+str(S)  # F://Shameless//Shameless_S1
    path = Path(url)
    # 如果文件夹不存在，则创建
    if not path.is_dir():
        os.mkdir(url)
    for i in range(len(m3u8_list)):
        print('正在下载第{}集...'.format(i+1))
        start = datetime.datetime.now().replace(microsecond=0)
        time.sleep(1)  # sleep一秒
        ts_urls = []  # 保存每一集的ts文件的真实url
        m3u8 = requests.get(url=m3u8_list[i])
        content = m3u8.text.split('\n')
        for s in content:
            if s.endswith('.ts'):
                ts_url = m3u8_list[i][:-10] + s.strip('\n') # 生成ts文件的真实url
                ts_urls.append(ts_url)
        download_ts(ts_urls,down_path=url+'//'+"E"+str(i+1)+'.ts')  # 根据ts的url下载每集的ts文件
        end = datetime.datetime.now().replace(microsecond=0)
        print('耗时：%s' % (end - start))
        print('第{}集下载完成...'.format(i+1))


# 根据ts下载链接下载文件
def download_ts(ts_urls,down_path):
    file = open(down_path, 'wb')
    for i in tqdm(range(len(ts_urls))):
        ts_url = ts_urls[i]  # 例:https://youku.com-youku.net/20180626/14084_f3588039/1000k/hls/80ed70a101f861.ts
        time.sleep(1)
        try:
            response = requests.get(url=ts_url, stream=True, verify=False)
            file.write(response.content)
        except Exception as e:
            print('异常请求：%s' % e.args)
    file.close()


if __name__ == '__main__':
    savefile_path = 'F://Shameless//'
    section_url = ['http://www.tv3w.com/dushiqinggan/wuchizhitudiyiji/5-1.html',
                   'http://www.tv3w.com/dushiqinggan/wuchizhitudierji/3-1.html',
                   'http://www.tv3w.com/dushiqinggan/wuchizhitudisanji/4-1.html',
                   'http://www.tv3w.com/dushiqinggan/wuchizhitudisiji/4-1.html',
                   'http://www.tv3w.com/dushiqinggan/wuchizhitudiwuji/7-1.html',
                   'http://www.tv3w.com/dushiqinggan/wuchizhitudiliuji/7-1.html',
                   'http://www.tv3w.com/dushiqinggan/wuchizhitudiqiji/6-1.html']
    for i in range(len(section_url)):
        print('开始下载第{}季...'.format(i+1))
        episode_url = get_m3u8_list(url=section_url[i],S=i+1) # 获取每季中每一集的m3u8地址
        download(episode_url,savefile_path,i+1)    # 下载每一季的ts文件并拼接
    print('done')
```

目标网站的资源前七季的源是同一个，这里只获取前七季。

## 改进

第一写完整的爬虫代码，可以改进的地方有很多：

1. 可以改为自动获取m3u8真实地址。
2. 使用多线程下载，提高下载速度。
3. 添加文件验证机制，确保下载正确。



#　参考链接

1. [Python爬虫——从流媒体网站获取的m3u8爬取视频](https://blog.csdn.net/kingyuan666/article/details/90247526)
2. [爬取m3u8视频](https://www.cnblogs.com/i-am-normal/p/11624225.html)