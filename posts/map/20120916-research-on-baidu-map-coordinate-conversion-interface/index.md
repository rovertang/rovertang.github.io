# 关于百度地图坐标转换接口的研究


这个世界的坐标系统已经让人搞得昏头转向(请看这篇：[国内各地图API比较](/posts/mapnavi/20120315-comparison-of-api-coordinate-systems-of-various-maps-in-china/))，而百度地图还用了自家的坐标系统，今天偶然看到百度地图批量转换接口，心想看看代码反转一下，但尝试无果。虽然将百度坐标转换成火星坐标不成，但我还是有些东西想和大家分享，使用百度地图坐标接口实现地球坐标转换到火星坐标。

在说这个问题之前，我们还是普及一下坐标系统的概念。我们使用GPS系统获得的坐标系统，基本为标准的国际通用的WGS84坐标系统，而我们的国测局出于安全考虑，推出了02坐标系统，就是在标准的WGS84坐标系统上进行了人为的偏移，并且还是非线性的，所有的导航软件导航地图都需要使用国家02坐标系统，比如Google地图、腾讯SOSO地图等就是直接使用了国家02坐标系统，我们有一个不成文的说法，前者叫地球坐标，后者叫火星坐标，并且，火星坐标是无法转换成地球坐标的(网上虽然有一定的方法，但基本上都是基于偏移数据库，精度较高的数据库需要购买，当然这都是一种破解手段)。而百度地图等，可能是出于商业化考虑，为了不让自己的用户流失，而推出了自家的坐标系统，就是谁也看不懂的百度坐标系统，在百度地图上，没有任何偏差，但你又无法将转换后的百度坐标系统反转回来，这样你用百度地图坐标就自然离不开百度了。对于这样的行径，很是无语，但作为一个公司的商业化手段，也算能理解。

说完坐标系统，我们自然能够知道这里的问题，我穷举了六个问题，来说说我对此的研究。

## 1、地球坐标转火星坐标

原则上，国家提供了保密插件，直接可算，但你必须通过正规的商业化渠道才可以获得，一般的个人是不可能从国测局取得保密插件代码的。

这个问题不是没有解决办法，因为网络地图公司就一定会使用到这样的接口，比如Google地图、MapABC地图等，网上一个朋友在iOS上实现了该转换，用的是高德MapABC的接口(详见[这里](http://www.keakon.net/2011/07/02/WGS84%E5%9D%90%E6%A0%87%E8%BD%AC%E7%81%AB%E6%98%9F%E5%9D%90%E6%A0%87%EF%BC%88iOS%E7%AF%87%EF%BC%89))，我来说说百度地图接口的做法。

接口地址：[http://api.map.baidu.com/ag/coord/convert?x=121.583140&amp;y=31.341174&amp;from=0&amp;to=2&amp;mode=1](http://api.map.baidu.com/ag/coord/convert?x=121.583140&amp;y=31.341174&amp;from=0&amp;to=2&amp;mode=1)

说明：x和y就是经纬度了，替换成你真实的经纬度即可，from和to表示坐标系，0表示地球坐标，2表示火星坐标，4表示百度坐标，所以这里是从地球坐标转换成火星坐标，mode参数未知。

结果：`[{&#34;error&#34;:0,&#34;x&#34;:&#34;MTIxLjU4NzM2NDA5NTA1&#34;,&#34;y&#34;:&#34;MzEuMzM5MDI3NTA2NTE=&#34;}]`

说明：error为0表示没有错误，返回的x和y是base64算法后的结果(可以自行Google加解密base64)，解密后就是：121.58736409505和31.33902750651，这个就是火星坐标。

问题：我不知道官方是否提供了这个方法，但验证下来基本没有偏差(第六位同MapABC加密出来的不同，原因未知)，第六位的偏差也可以基本忽略。

本想用这个接口自己来写一个小软件的，但想想过于麻烦，所以有心的朋友来写一个吧，当然，也要注意，这个接口的调用最好是异步的，并且每次最多好像是20个。

## 2、火星坐标转地球坐标

如上文所述，国家是不可能公开这个算法的，网上流传的基本上都是基于数据库的，而高精度的反算数据库有人是卖钱的。

关于这方面的研究，三年前就已经是热火朝天了，只是个人有一两个工具可用，所以也一直无心研究具体实现。至于数据库，0.1的数据库应该是比较容易获得的，由于手头看到的都已经加密成二进制，所以待我找到后再同大家分享吧。顺便推荐一下这篇：[一种根据纠偏数据对火星坐标进行完美拟合的方法](http://blog.sina.com.cn/s/blog_538036cf0100pxbl.html)，有兴趣的朋友可以研究一下，或者做成一个工具。

## 3、地球坐标到百度坐标

百度已经提供了两个示例：

坐标转换示例：[http://dev.baidu.com/wiki/static/map/API/examples/?v=1.3&amp;0_6#0&amp;6](http://dev.baidu.com/wiki/static/map/API/examples/?v=1.3&amp;0_6#0&amp;6)

批量坐标转换示例：[http://dev.baidu.com/wiki/static/map/API/examples/?v=1.3&amp;0_7#0&amp;7](http://dev.baidu.com/wiki/static/map/API/examples/?v=1.3&amp;0_7#0&amp;7)

虽然这两个示例中，百度提供了一个[js](http://dev.baidu.com/wiki/static/map/API/examples/script/changeMore.js)，但实际上也逃离不了第一点中描述的接口[http://api.map.baidu.com/ag/coord/convert?x=121.583140&amp;y=31.341174&amp;from=0&amp;to=2&amp;mode=1](http://api.map.baidu.com/ag/coord/convert?x=121.583140&amp;y=31.341174&amp;from=0&amp;to=2&amp;mode=1)，只是变更为了from 0 to 4。以此类推，下述第四点即为from 2 to 4。

## 4、火星坐标到百度坐标

同第三点所述。

## 5、百度坐标到火星坐标

这是我本次所想破解的问题，结合上文所述，地球坐标到火星坐标是国家的方法，那么火星坐标到百度坐标应该是自己的算法，既然搜狗能够解密出百度的坐标(提供的也仅仅是接口，无实际算法)，那么按照道理根据规律也是可以进行解密。我做了几个坐标，尝试着看出其中的规律：

| 火星经度            | 火星纬度 | 百度经度(base64)         | 百度纬度(base64)         | 百度经度                | 百度纬度               |
|-----------------|------|----------------------|----------------------|---------------------|--------------------|
| 121             | 31   | MTIxLjAwNjU2MzI3ODQ2 | MzEuMDA1ODIyNzk4NjUz | 121.006563278460 | 31.005822798653 |
| 122             | 32   | MTIyLjAwNjUzMTI0NjE= | MzIuMDA1ODEyNjA1NDk0 | 122.006531246100 | 32.005812605494 |
| 123             | 33   | MTIzLjAwNjQwMDk5OTQ1 | MzMuMDA2MzY4OTk5ODUx | 123.006400999450 | 33.006368999851 |
| 124             | 34   | MTI0LjAwNjU2NzcwMzgy | MzQuMDA1ODE4NTgwMTIy | 124.006567703820 | 34.005818580122 |
| 121 | 31   | MTIxLjAwNjU2MzI3ODQ3 | MzEuMDA1ODIyNzk4NjUz | 121.006563278470 | 31.005822798653 |
| 121   | 31   | MTIxLjAwNjU2MzI3OTQ2 | MzEuMDA1ODIyNzk4NjM3 | 121.006563279460 | 31.005822798637 |

可惜我不是学数学的，对于非线性的分析确实很难，只好作罢。不知道有大侠可否分析出其中的规律来？很是期待。

## 6、百度坐标到地球坐标

这个问题基本无解，即便有解也需要先解决第五点和第二点。这里就不多说了。

这就是我一个晚上对此问题的研究，欢迎大家继续研究讨论。

本文飞书文档：[关于百度地图坐标转换接口的研究](https://rovertang.feishu.cn/docx/doxcnkKdDVTDj8sBGTu141MVdWg)


---

> 作者: [RoverTang](https://rovertang.com)  
> URL: https://blog.rovertang.com/posts/map/20120916-research-on-baidu-map-coordinate-conversion-interface/  

