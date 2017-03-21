#TianYanCha_Spider
一个爬取[天眼查](http://www.tianyancha.com)的企业信息的爬虫，遍历机制是根据企业名称进行搜索，企业名称保存在`name_list.txt`中，来自[黄页88](http://www.huangye88.com/)，其爬虫代码见[Yellow_Page](https://github.com/Range0122/yellow_page)。
# Scrapy + Selenium + Phantomjs
由于目标网页中需要抓取的数据采用了JS渲染，通过普通的request无法抓到，所以使用了`selenium` + `phantomjs`
目前只是做了在搜索了企业名称之后对于展示了企业详细信息的网站的url的获取，后续将更新抓取详细信息的代码。
#更新
##17.03.20
因为浏览器环境对内存和CPU的消耗都非常严重，模拟浏览器环境的爬虫代码要尽可能避免，相比之下，效率更高的方式是通过分页网页的代码，找到网页请求得到数据的url，可以通过伪装header的方式直接从源取得数据，通常是以json的格式返回数据。
分析天眼查的网页，发现有几个数据包都是通过json返回回来的，非常简洁整齐，简直让爬虫垂涎三次。然后发现**cookies**里面有*_utm*、*token*、*paaptp*这三个参数每次都会变，然而根本找不出它们之间的算法，即使原模原样包装好header也获取不到json数据。
在网上找了很多关于天眼查爬虫的列子，一月份的帖子url跟现在的已经有了区别，中间加了`v2`应该是更新过了反爬虫机制 ，意外还发现他们在招反爬虫工程师。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4218178-b4795016379278ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
所以没办法还是先用`selenium`+`phantomjs`，虽然效率很低，但还是先把事情做成了再考虑后续优化的事情吧。
另一方面，天眼查访问次数稍微多一点，就会出现滑动验证码，所以还要改ip代理，不得不说这是我目前见过的反爬虫机制最强的一个网站了。
真是一场`（月薪 -1.5K VS 15K+）`的较量。
遇到一个很奇怪的问题，我之前使用`Scrapy-Splash`渲染JS的时候也遇到过。就是爬取的JS的数据很不稳定，有时候能抓到JS数据，有时候又莫名其妙根本拿不到，之前我一直以为是`Slash`自身的问题，结果发现`selenium`+`phantomjs`现在也出了这种问题，并且尝试多次都莫名其妙抓不到数据了。没办法只好回过头去尝试从[国家企业信用信息公示网站](http://www.gsxt.gov.cn/index.html)，结果更强的是url中根本不包含searchword，每次搜索都有滑动验证码，虽然github上有现成的破解代码，但是成功率也不是特别理想。另外每次post都会传geetest挑战值什么的，尝试直接在打开url之后再请求，爬取结果直接返回**403 forbidden**

#17.03.21
趁着早上解决了一些怪问题，赶快传一个终于能跑起来的上来。
* JS抓不到数据的问题，网页应该是先加载基本元素再请求JS，所以如果抓取速度太快根本来不及渲染JS就只把基本的html返回了，解决方法是在`driver.get(url)`之后`time.sleep(10)`，等待JS加载完成之后再获取html，但这样做会导致效率低一些。
* 跑的时候每次都会出现`KeyError`，仔细查看了 `item`和`pipeline`之后并没有发现任何拼写上的问题，最初以为是`ItemLoader`的问题，然后又去读了官方文档，并没有任何错误，但是每次跑程序的时候死活都会报错。最后发现是Item里面少了括号。`Scrapy.Field()` **而不是** `Scrapy.Field`。细节出错简直 *drove me crazy
* 修改了MiddleWare，选用PhantomJS来解析Request，并且这个网站还有一个反爬虫的机制：因为PhantomJS这样的无界面浏览器除了程序员基本没什么人用，所以如不过进行`User-Agent`伪装的话，网站就会返回一个随机的错误的数据回来。
* 接下来就是上Tor改IP代理，天眼查对IP的数据流量还是比较敏感的，稍不注意就直接上验证码封IP，实在是不想搞geetest验证码。然后就是继续补充完相应的数据条目，整体来看还是比较复杂的，部分数据的翻页也是很棘手的一个事情。