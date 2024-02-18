---
title: Python爬虫入门
date: 2020-02-02 20:41:59
tags: 
    - Python
    - 爬虫
    - 多线程
    - 观察者模式
categories: Python初级应用
id: Python-crawlers-beginning
---
# 前言
在初步掌握Python基本语法后，这里使用Py进行一个简单的爬取图片应用作为入门训练。  
首先爬虫的基本步骤可以概括为：

- 输入域名 -> 下载源代码 -> 分析图片路径 -> 下载图片

这里我们选取[http://www.meizitu.com/a/pure.html](http://www.meizitu.com/a/pure.html)作为爬取的目标网站。因为这个网站好分析，爬取简单。

<!--more-->
# 分析页面
![1tR0mD.jpg](https://s2.ax1x.com/2020/02/02/1tR0mD.jpg)

爬虫最重要的一点，便是寻找到分页地点。因为分页处代表可能存在规律可循。建立在既有规律的基础上，我们可以更智能、准确、方便得爬取到相关数据

以本网站为例，我们可从上述图片中发现分页：  
![1tTWwV.jpg](https://s2.ax1x.com/2020/02/02/1tTWwV.jpg)

如图所示，使用浏览器的开发者工具可发现分页规律：

    http://www.meizitu.com/a/pure_1.html
    http://www.meizitu.com/a/pure_2.html
    http://www.meizitu.com/a/pure_3.html
    http://www.meizitu.com/a/pure_4.html

接下来我们使用Python实现这部分（采用面向对象的思想）

```python
import requests
all_urls = []   #我们拼接好的图片集和列表路径
class Spider():
    #构造函数，初始化数据使用
    def __init__(self, target_url, headers):
        self.target_url =  target_url
        self.headers = headers

    #获取所有的想要爬取的URL
    def getUrls(self, start_page, page_num):
        global all_urls
        #循环得到URL
        for i in range(start_page, page_num + 1):
            url = self.target_url % i   #注意这里并非取余操作，而是格式化字符串
            all_urls.append(url)

if __name__ == "__main__":
    headers = {
        # 这部分在浏览器中输入 about:version查看
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.62 Safari/537.36',
        'HOST': 'www.meizitu.com'
    }
    target_url = 'http://www.meizitu.com/a/pure_%d.html'    #图片集和列表规则

    spider = Spider(target_url, headers)
    spider.getUrls(1,16)
    print(all_urls)
```
以上的代码主要有以下几个要点：

- 声明了一个`class Spider()`类，并使用`def __init__`声明一个构造函数  
- 拼接URL，我们可以选择很多方法，这里采用最简单直接的字符串拼接方法  
- 注意上述代码中有一个全局变量`all_urls`，用来存储我们的所有分页的URL  

# 爬虫核心

我们需要分析页面中的逻辑。首先打开[http://www.meizitu.com/a/pure_1.html](http://www.meizitu.com/a/pure_1.html)，右键审查但愿元素：  
![1UCMVS.jpg](https://s2.ax1x.com/2020/02/03/1UCMVS.jpg)  
![1UiY7T.jpg](https://s2.ax1x.com/2020/02/03/1UiY7T.jpg)  
上图红色框内的链接即为所需要的链接  
点击图片后，可发现进入图片详情页面，发现尽然是套图，那么现在的问题在于：  
首先，我们需要在 http://www.meizitu.com/a/pure_1.html 这种页面中爬取所有的 http://www.meizitu.com/a/5585.html 这种地址  
这里我们采用多线程的方案进行爬取，此外还涉及到名为“观察者模式”的设计模式。  
首先引入三个模块，分别为多线程、正则表达式、时间模块。并新增一个全局的变量，由于是多线程操作，这里需要引入线程锁：

    import threading	#多线程模块
    import re			#正则表达式模块
    import time			#时间模块
    
    all_img_urls = []	#图片列表页面的数组
    g_lock = threading.Lock()	#初始化一个锁

下面声明一个生产者的类，用来不断获取详情页地址，然后添加到`all_img_urls`这个全局变量中

```python
#生产者，负责从每个页面提取图片列表链接
class Producer(threading.Thread):   

    def run(self):
        headers = {
            # 这部分在浏览器中输入 about:version查看
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.62 Safari/537.36',
            'HOST': 'www.meizitu.com'
        }
        global all_urls
        while len(all_urls) > 0 :
            g_lock.acquire()  #在访问all_urls的时候，需要使用锁机制
            page_url = all_urls.pop()   #通过pop方法移除最后一个元素，并且返回该值
            
            g_lock.release() #使用完成之后及时把锁给释放，方便其他线程使用
            try:
                print("分析"+page_url)   
                response = requests.get(page_url , headers = headers,timeout=3)
                all_pic_link = re.findall('<a target=\'_blank\' href="(.*?)">',response.text,re.S)   
                global all_img_urls
                g_lock.acquire()   #这里还有一个锁
                all_img_urls += all_pic_link   #这个地方注意数组的拼接，没有用append直接用的+=也算是python的一个新语法吧
                print(all_img_urls)
                g_lock.release()   #释放锁
                time.sleep(0.5)
            except:
                pass
```

- 上述代码从threading.Thread中继承了一个子类  
- 线程锁：在上述代码中，当我们操作`all_urls.pop()`时，我们不希望其他线程对其进行同时操作，否则可能发生意外。所以我们使用`g_lock.acquire()`锁定资源，然后使用完成之后，记住一定要立即释放`g_lock.release()`，否则此资源被一直占用，程序无法继续进行。  
- 匹配网页中的URL，这里使用正则表达式，后面我们会使用其它方法进行匹配。  
`re.findall()`方法是获取所有匹配到的内容  
- 这里使用`try: except`包装可能出现异常的代码

如果上述代码均没有出现问题，那么我们可以在程序的入口处编写以下代码来执行程序

	for x in range(2):
		t = Producer()
		t.start()

因为我们的Producer继承自threading.Thread类。所以我们必须实现的一个方法是`def run`（已在上述的代码中呈现）。运行结果如下图所示：  
![1B0XvV.jpg](https://s2.ax1x.com/2020/02/04/1B0XvV.jpg)
接下来，我们需要执行这样一步操作，我想要等待图片详情页面全部获取完毕，在进行接下来的分析操作。  
这里增加代码：

    #threads= []   
    #开启两个线程去访问
    for x in range(2):
    t = Producer()
    t.start()
    #threads.append(t)
    
    # for tt in threads:
    #     tt.join()
    
    print("进行到我这里了")

注释关键代码，运行如下：

    [linuxboy@bogon demo]$ python3 down.py
    分析http://www.meizitu.com/a/pure_2.html
    分析http://www.meizitu.com/a/pure_1.html
    进行到我这里了
    ['http://www.meizitu.com/a/5585.html',

把上面的tt.join等代码注释打开，运行如下：

    [linuxboy@bogon demo]$ python3 down.py
    分析http://www.meizitu.com/a/pure_2.html
    分析http://www.meizitu.com/a/pure_1.html
    ['http://www.meizitu.com/a/5429.html', ......
    进行到我这里了

发现一个本质的区别，就是，我们由于是多线程的程序，所以，当程序跑起来之后，`print("进行到我这里了")`不会等到其他线程结束，就会运行到，但是当我们改造成上面的代码之后，也就是加入了关键的代码 `tt.join()` 那么主线程的代码会等到所以子线程运行完毕之后，在接着向下运行。这就满足了，我刚才说的，先获取到所有的图片详情页面的集合，这一条件了。

join所完成的工作就是线程同步，即主线程遇到join之后进入阻塞状态，一直等待其他的子线程执行结束之后，主线程在继续执行。这个大家在以后可能经常会碰到。

下面编写一个消费者/观察者，也就是不断关注刚才我们获取的那些图片详情页面的数组。

添加一个全局变量，用来存储获取到的图片链接  
`pic_links = []            #图片地址列表`

```python
#消费者
class Consumer(threading.Thread) : 
    def run(self):
        headers = {
            'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0',
            'HOST':'www.meizitu.com'
        }
        global all_img_urls   #调用全局的图片详情页面的数组
        print("%s is running " % threading.current_thread)
        while len(all_img_urls) >0 : 
            g_lock.acquire()
            img_url = all_img_urls.pop()
            g_lock.release()
            try:
                response = requests.get(img_url , headers = headers )
                response.encoding='gb2312'   #由于我们调用的页面编码是GB2312，所以需要设置一下编码
                title = re.search('<title>(.*?) | 妹子图</title>',response.text).group(1)
                all_pic_src = re.findall('<img alt=.*?src="(.*?)" /><br />',response.text,re.S)
                
                pic_dict = {title:all_pic_src}   #python字典
                global pic_links
                g_lock.acquire()
                pic_links.append(pic_dict)    #字典数组
                print(title+" 获取成功")
                g_lock.release()
                
            except:
                pass
            time.sleep(0.5)
```

代码中比较重要的一些部分，我已经使用注释写好了，大家可以直接参考。大家一定要注意我上面使用了两个正则表达式，分别用来匹配title和图片的url这个title是为了后面创建不同的文件夹使用的，所以大家注意吧。

    #开启10个线程去获取链接
    for x in range(10):
    	ta = Consumer()
    	ta.start()

运行结果：

    [linuxboy@bogon demo]$ python3 down.py
    分析http://www.meizitu.com/a/pure_2.html
    分析http://www.meizitu.com/a/pure_1.html
    ['http://www.meizitu.com/a/5585.html', ......
    <function current_thread at 0x7f7caef851e0> is running 
    <function current_thread at 0x7f7caef851e0> is running 
    <function current_thread at 0x7f7caef851e0> is running 
    <function current_thread at 0x7f7caef851e0> is running 
    <function current_thread at 0x7f7caef851e0> is running 
    <function current_thread at 0x7f7caef851e0> is running 
    <function current_thread at 0x7f7caef851e0> is running 
    <function current_thread at 0x7f7caef851e0> is running 
    进行到我这里了
    <function current_thread at 0x7f7caef851e0> is running 
    <function current_thread at 0x7f7caef851e0> is running 
    清纯美如画，摄影师的御用麻豆 获取成功
    宅男女神叶梓萱近日拍摄一组火爆写真 获取成功
    美（bao）胸（ru）女王带来制服诱惑 获取成功
    每天睁开眼看到美好的你，便是幸福 获取成功
    可爱女孩，愿暖风呵护纯真和执着 获取成功
    清纯妹子如一缕阳光温暖这个冬天 获取成功 .....

距离成功有进了一大步

接下来就是，我们开篇提到的那个存储图片的操作了，还是同样的步骤，写一个自定义的类

```python
class DownPic(threading.Thread) :

    def run(self):
        headers = {
            'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0',
            'HOST':'mm.chinasareview.com'
        
        }
        while True:   #  这个地方写成死循环，为的是不断监控图片链接数组是否更新
            global pic_links
            # 上锁
            g_lock.acquire()
            if len(pic_links) == 0:   #如果没有图片了，就解锁
                # 不管什么情况，都要释放锁
                g_lock.release()
                continue
            else:
                pic = pic_links.pop()
                g_lock.release()
                # 遍历字典列表
                for key,values in  pic.items():
                    path=key.rstrip("\\")
                    is_exists=os.path.exists(path)
                    # 判断结果
                    if not is_exists:
                        # 如果不存在则创建目录
                        # 创建目录操作函数
                        os.makedirs(path) 
                
                        print (path+'目录创建成功')
                        
                    else:
                        # 如果目录存在则不创建，并提示目录已存在
                        print(path+' 目录已存在') 
                    for pic in values :
                        filename = path+"/"+pic.split('/')[-1]
                        if os.path.exists(filename):
                            continue
                        else:
                            response = requests.get(pic,headers=headers)
                            with open(filename,'wb') as f :
                                f.write(response.content)
                                f.close

```

我们获取图片链接之后，就需要下载了，我上面的代码是首先创建了一个之前获取到title的文件目录，然后在目录里面通过下面的代码,去创建一个文件。

涉及到文件操作，引入一个新的模块  
`import os  #目录操作模块`

```python
# 遍历字典列表
for key,values in  pic.items():
    path=key.rstrip("\\")
    is_exists=os.path.exists(path)
    # 判断结果
    if not is_exists:
        # 如果不存在则创建目录
        # 创建目录操作函数
        os.makedirs(path) 

        print (path+'目录创建成功')
        
    else:
        # 如果目录存在则不创建，并提示目录已存在
        print(path+' 目录已存在') 
    for pic in values :
        filename = path+"/"+pic.split('/')[-1]
        if os.path.exists(filename):
            continue
        else:
            response = requests.get(pic,headers=headers)
            with open(filename,'wb') as f :
                f.write(response.content)
                f.close
```

因为我们的图片链接数组，里面存放是的字典格式，也就是下面这种格式

	[{"妹子图1":["http://mm.chinasareview.com/wp-content/uploads/2016a/08/24/01.jpg","http://mm.chinasareview.com/wp-content/uploads/2016a/08/24/02.jpg"."http://mm.chinasareview.com/wp-content/uploads/2016a/08/24/03.jpg"]},{"妹子图2":["http://mm.chinasareview.com/wp-content/uploads/2016a/08/24/01.jpg","http://mm.chinasareview.com/wp-content/uploads/2016a/08/24/02.jpg"."http://mm.chinasareview.com/wp-content/uploads/2016a/08/24/03.jpg"]},{"妹子图3":["http://mm.chinasareview.com/wp-content/uploads/2016a/08/24/01.jpg","http://mm.chinasareview.com/wp-content/uploads/2016a/08/24/02.jpg"."http://mm.chinasareview.com/wp-content/uploads/2016a/08/24/03.jpg"]}]

需要先循环第一层,获取title，创建目录之后，在循环第二层去下载图片，代码中，我们在修改一下，把异常处理添加上。
	
	try:
	    response = requests.get(pic,headers=headers)
	    with open(filename,'wb') as f :
	        f.write(response.content)
	        f.close
	except Exception as e:
	    print(e)
	    pass

然后在主程序中编写代码

	#开启10个线程保存图片
	for x in range(10):
	    down = DownPic()
	    down.start()

运行结果：


	[linuxboy@bogon demo]$ python3 down.py
	分析http://www.meizitu.com/a/pure_2.html
	分析http://www.meizitu.com/a/pure_1.html
	['http://www.meizitu.com/a/5585.html', 'http://www.meizitu.com/a/5577.html', 'http://www.meizitu.com/a/5576.html', 'http://www.meizitu.com/a/5574.html', 'http://www.meizitu.com/a/5569.html', .......
	<function current_thread at 0x7fa5121f2268> is running 
	<function current_thread at 0x7fa5121f2268> is running 
	<function current_thread at 0x7fa5121f2268> is running 
	进行到我这里了
	清纯妹子如一缕阳光温暖这个冬天 获取成功
	清纯妹子如一缕阳光温暖这个冬天目录创建成功
	可爱女孩，愿暖风呵护纯真和执着 获取成功
	可爱女孩，愿暖风呵护纯真和执着目录创建成功
	超美，纯纯的你与蓝蓝的天相得益彰 获取成功
	超美，纯纯的你与蓝蓝的天相得益彰目录创建成功
	美丽冻人，雪地里的跆拳道少女 获取成功
	五官精致的美眉，仿佛童话里的公主 获取成功
	有自信迷人的笑容，每天都是灿烂的 获取成功
	五官精致的美眉，仿佛童话里的公主目录创建成功
	有自信迷人的笑容，每天都是灿烂的目录创建成功
	清纯美如画，摄影师的御用麻豆 获取成功

文件目录下面同时出现

![1BB0Gn.jpg](https://s2.ax1x.com/2020/02/04/1BB0Gn.jpg)

点开一个目录

![1BBB2q.jpg](https://s2.ax1x.com/2020/02/04/1BBB2q.jpg)

最后我们在代码的头部写上  
`# -*- coding: UTF-8 -*- `以防出现`Non-ASCII character 'xe5' in file`报错问题

# 完整代码

```python
import requests
import threading  # 多线程模块
import re  # 正则表达式模块
import time  # 时间模块
import os  # 目录操作模块

all_urls = []  # 我们拼接好的图片集和列表路径
all_img_urls = []  # 图片列表页面的数组
pic_links = []  # 图片地址列表
g_lock = threading.Lock()  # 初始化一个锁


class Spider():
    # 构造函数，初始化数据使用
    def __init__(self, target_url, headers):
        self.target_url = target_url
        self.headers = headers

    # 获取所有的想要抓取的URL
    def getUrls(self, start_page, page_num):
        # 循环得到URL
        for i in range(start_page, page_num + 1):
            url = self.target_url % i
            all_urls.append(url)


# 消费者
class Consumer(threading.Thread):
    def run(self):
        headers = {
            'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0',
            'HOST': 'www.meizitu.com'
        }
        global all_img_urls
        print("%s is running " % threading.current_thread)
        while len(all_img_urls) > 0:
            g_lock.acquire()
            img_url = all_img_urls.pop()
            g_lock.release()
            try:
                response = requests.get(img_url, headers=headers)

                response.encoding = 'gb2312'
                title = re.search('<title>(.*?) | 妹子图</title>', response.text).group(1)
                all_pic_src = re.findall('<img alt=.*?src="(.*?)" /><br />', response.text, re.S)

                pic_dict = {title: all_pic_src}
                global pic_links
                g_lock.acquire()
                pic_links.append(pic_dict)
                print(title + " 获取成功")
                g_lock.release()

            except:
                pass
            time.sleep(0.5)


# 生产者，负责从每个页面提取图片列表链接
class Producer(threading.Thread):
    def run(self):
        headers = {
            'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0',
            'HOST': 'www.meizitu.com'
        }
        global all_urls
        while len(all_urls) > 0:
            g_lock.acquire()  # 在访问all_urls的时候，需要使用锁机制
            page_url = all_urls.pop()  # 通过pop方法移除最后一个元素，并且返回该值

            g_lock.release()  # 使用完成之后及时把锁给释放，方便其他线程使用
            try:
                print("分析" + page_url)
                response = requests.get(page_url, headers=headers, timeout=3)
                all_pic_link = re.findall('<a target=\'_blank\' href="(.*?)">', response.text, re.S)
                global all_img_urls
                g_lock.acquire()
                all_img_urls += all_pic_link
                print(all_img_urls)
                g_lock.release()

            except:
                pass
            time.sleep(0.5)


class DownPic(threading.Thread):
    def run(self):
        headers = {
            'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0',
            'HOST': 'mm.chinasareview.com',
            'Cookie': 'safedog-flow-item=; UM_distinctid=1634d30879a9-06230f37a83b4b8-38694646-1fa400-1634d30879c451; CNZZDATA30056528=cnzz_eid%3D1182386559-1526005899-http%253A%252F%252Fwww.meizitu.com%252F%26ntime%3D1526022264; bdshare_firstime=1526025336868'
        }
        while True:
            global pic_links
            # 上锁
            g_lock.acquire()
            if len(pic_links) == 0:
                # 不管什么情况，都要释放锁
                g_lock.release()
                continue
            else:
                pic = pic_links.pop()
                g_lock.release()
                # 遍历字典列表
                for key, values in pic.items():
                    path = key.rstrip("\\")
                    is_exists = os.path.exists(path)
                    # 判断结果
                    if not is_exists:
                        # 如果不存在则创建目录
                        # 创建目录操作函数
                        os.makedirs(path)

                        print(path + '目录创建成功')

                    else:
                        # 如果目录存在则不创建，并提示目录已存在
                        print(path + ' 目录已存在')
                    for pic in values:
                        filename = path + "/" + pic.split('/')[-1]
                        if os.path.exists(filename):
                            continue
                        else:
                            try:
                                response = requests.get(pic, headers=headers)
                                with open(filename, 'wb') as f:
                                    f.write(response.content)
                                    f.close
                            except Exception as e:
                                print(e)
                                pass


if __name__ == "__main__":
    headers = {
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0',
        'HOST': 'www.meizitu.com'
    }
    target_url = 'http://www.meizitu.com/a/pure_%d.html'  # 图片集和列表规则

    spider = Spider(target_url, headers)
    spider.getUrls(1, 2)

    threads = []
    # 开启两个线程去访问
    for x in range(2):
        t = Producer()
        t.start()
        threads.append(t)

    for tt in threads:
        tt.join()
    # 开启10个线程去获取链接
    for x in range(10):
        ta = Consumer()
        ta.start()

    # 开启5个线程保存图片
    for x in range(10):
        down = DownPic()
        down.start()

    print("进行到我这里了")
```
# 他解

我们可以知道，同一个页面需要爬取的内容可能具有多方面的特征，上述的获取所需网页链接的方案因此不唯一。

可以发现图片链接还有以下的特征可以提取：

![1BDqXV.jpg](https://s2.ax1x.com/2020/02/04/1BDqXV.jpg)  


![1BDOmT.jpg](https://s2.ax1x.com/2020/02/04/1BDOmT.jpg)


因此可以考虑采用以下代码进行爬取：

```python
import requests
import os
import time
import threading
from bs4 import BeautifulSoup


def download_page(url):
    '''
    用于下载页面
    '''
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36 Edge/18.18362"}
    r = requests.get(url, headers=headers)
    r.encoding = 'gb2312'
    return r.text


def get_pic_list(html):
    '''
    获取每个页面的套图列表,之后循环调用get_pic函数获取图片
    '''
    soup = BeautifulSoup(html, 'html.parser')
    pic_list = soup.find_all('li', class_='wp-item')
    for i in pic_list:
        a_tag = i.find('h3', class_='tit').find('a')
        link = a_tag.get('href')
        text = a_tag.get_text()
        get_pic(link, text)


def get_pic(link, text):
    '''
    获取当前页面的图片,并保存
    '''
    html = download_page(link)  # 下载界面
    soup = BeautifulSoup(html, 'html.parser')
    pic_list = soup.find('div', id="picture").find_all('img')  # 找到界面所有图片
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36 Edge/18.18362"}
    create_dir('pic/{}'.format(text))
    for i in pic_list:
        pic_link = i.get('src')  # 拿到图片的具体 url
        r = requests.get(pic_link, headers=headers)  # 下载图片，之后保存到文件
        with open('pic/{}/{}'.format(text, pic_link.split('/')[-1]), 'wb') as f:
            f.write(r.content)
            time.sleep(3)   # 休息一下，不要给网站太大压力，避免被封


def create_dir(name):
    if not os.path.exists(name):
        os.makedirs(name)


def execute(url):
    page_html = download_page(url)
    get_pic_list(page_html)


def main():
    create_dir('pic')
    queue = [i for i in range(1, 72)]   # 构造 url 链接 页码。
    threads = []
    while len(queue) > 0:
        for thread in threads:
            if not thread.is_alive():
                threads.remove(thread)
        while len(threads) < 5 and len(queue) > 0:   # 最大线程数设置为 5
            cur_page = queue.pop(0)
            url = 'http://meizitu.com/a/more_{}.html'.format(cur_page)
            thread = threading.Thread(target=execute, args=(url,))
            thread.setDaemon(True)
            thread.start()
            print('{}正在下载{}页'.format(threading.current_thread().name, cur_page))
            threads.append(thread)


if __name__ == '__main__':
    main()
```

