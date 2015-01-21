---
layout: post
date: 2013-09-02 09:18:00
title: 虎扑翻译团数据分析
categories: [Programming,cn]
tags: [life,hupu,translation]
---
这个其实是一时兴起做的一些东西，之前是因为每周统计翻译团的作品，所以为了省事，写了个小脚本来抓取每周翻译团lounge翻译出来的作品，后来就想到其实可以在此基础上对翻译团所有历史翻译的作品抓下来做点分析(这就是计算机的好处了，做一次和做一百次对你来说可能是很麻烦的事情，但是对电脑来说只是个时间的问题)。简单说一些这里抓取的原则，大家都知道翻译团的帖子审稿完成移出去之后都会标成蓝色，所以我抓取的就是翻译团lounge版面里面所有被标成蓝色的帖子。


## overall statistics




这里的数据一共收录了12949篇文章，当时收集的时候主要是根据文章的加色来简单判断的，所以其中难免存在一些多余或者是遗漏（特别是早期的一些文章加色上还没有十分统一)，但我想信这对总体的数据影响还属于可以接受的程度。

这里的数据已经全部传到了[google doc](https://docs.google.com/spreadsheet/ccc?key=0ArfSjsQRWyiJdGN5M0NOc0M5MlNUVFYxRzVMLWwzSlE&usp=sharing)里面，欢迎大家一起研究分析

### 一些事实

* 这里收录的最早的一篇文章 [资源集束器][first],其实这个并不是翻译帖子，不过当时也被加了蓝色，所以就被收进来了。

[first]: http//bbs.hupu.com/48786.html   

* 12949篇帖子一共有1947个ID,平均每个id发帖6.65篇,标准差为10.74101,可以看出其实产量水平的差异还是很大的。

* 在所有的id中，发帖前十的十大id是：

<table border="0">
<tr>
<td>id</td><td>作品数</td></tr>
<tr>
<td>yychosenone</td><td>120</td></tr>
<tr>
<td>rayallen9</td><td>114</td></tr>
<tr>
<td>andrew_nankai</td><td>97</td></tr>
<tr>
<td>foxlala</td><td>88</td></tr>
<tr>
<td>tristan7</td><td>88</td></tr>
<tr>
<td>cnDARk.WS7LP</td><td>85</td></tr>
<tr>
<td>常常有人问我</td><td>84</td></tr>
<tr>
<td>elgin1234</td><td>81</td></tr>
<tr>
<td>wczlwczl</td><td>69</td></tr>
<tr>
<td>月残梦晓</td><td>65</td></tr>
</table>
![plot of chunk unnamed-chunk-2](/media/img/top.png) 


* 前100名的译者一共翻译了4387篇帖子，占到了全部作品的33.88%，超过了1/3, 和著名的2/8定律相比，前20%的译者翻译了8752篇，占到全部帖子的66%
![plot of chunk unnamed-chunk-3](/media/img/distribute.png) 


* 时间跨度最长的译者是：
[LAL](http://my.hupu.com/599)他的第一篇译作可以追溯到[2005年](http://bbs.hupu.com/52564.html)，而他在[2012年](http://bbs.hupu.com/3837073.html)依然还曾经有过作品产出，是所有译者里面时间跨度最长（单以发帖而论）

* 单月翻译产出的最高记录是： 2008年8月份的 [rayallen9](http://my.hupu.com/rayallen9)贡献了45篇作品

* 历年翻译作品冠军
<table border="0">
<tr>
<td></td><td>id</td><td>作品数</td></tr>
<tr>
<td>2005</td><td>q412</td><td>25</td></tr>
<tr>
<td>2006</td><td>sprite</td><td>52</td></tr>
<tr>
<td>2007</td><td>wczlwczl</td><td>63</td></tr>
<tr>
<td>2008</td><td>rayallen9</td><td>91</td></tr>
<tr>
<td>2009</td><td>andrew_nankai</td><td>48</td></tr>
<tr>
<td>2010</td><td>Lemon-007</td><td>37</td></tr>
<tr>
<td>2011</td><td>yychosenone</td><td>75</td></tr>
<tr>
<td>2012</td><td>小哥ralf</td><td>55</td></tr>
<tr>
<td>2013</td><td>meadsun34</td><td>30</td></tr>
</table>


* 月度数据的趋势图

![plot of chunk unnamed-chunk-5](/media/img/monthly-trend.png) 



* 年度数据的图

![plot of chunk unnamed-chunk-6](/media/img/yearly.png) 


* 每月1-31日每天的历史总产量

![plot of chunk unnamed-chunk-7](/media/img/daily.png) 



* 标题最长的......
[Berry Tramel: 续约 Kevin Durant 可以使Sam Presti 继续留在俄城](http://bbs.hupu.com/1414959.html)



上面都是一些基本的数据分析，当然由于数据本身的简单性，也做不了太复杂的分析，接下来要做的工作主要是一些动态的展示，初步的构想是：

* 按找译者作品数的气泡图
* 时间轴的作品展示
* 总数的motion chart

今天就先写这么多吧，改天有时间，再继续后面的分析。
