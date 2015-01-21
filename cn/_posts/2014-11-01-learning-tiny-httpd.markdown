---
layout: post
date: 2014-11-01 21:38:21
title: tiny httpd webserver 学习
categories: [Programming,cn]
tags: [C++,服务器，Linux]
---


最近刷算法题目，顿觉自己的C++知识极度匮乏（其实本来就没怎么学会），所以打算借刷算法题目的时候顺便提高一下自己的C++能力，毕竟我现在的感觉，算法和编程其实还是两种比较不同的能力，当然这个可能要放到另外的一篇文章里面来讲的话题了。

总之就是，我在伯乐上看到一篇不错的文章[最值得阅读学习的 10 个 C 语言开源项目代码] [bole],
打算挑几个学习一下，最感兴趣的当然是超轻量级的[Tinyhttpd Server](http://sourceforge.net/projects/tinyhttpd/)了，全部代码居然只有502行，学习起来的难度应该相对小一些。

核心的服务器文件只有一个 httpd.c,除了main函数之外只有如下几个函数
	
	void accept_request(int);
	void bad_request(int);
	void cat(int, FILE *);
	void cannot_execute(int);
	void error_die(const char *);
	void execute_cgi(int, const char *, const char *, const char *);
	int get_line(int, char *, int);
	void headers(int, const char *);
	void not_found(int);
	void serve_file(int, const char *);
	int startup(u_short *);
	void unimplemented(int);

可以看到其中两个函数*execute_cgi*和*serve_file*分别对应静态文件和动态文件处理，[CGI](http://en.wikipedia.org/wiki/Common_Gateway_Interface)当然就是用来动态生成内容的脚本，静态文件就是各种文本（html).

下面逐个看一下各个函数的实现：

首先是几个辅助函数：

	get_line:
	int get_line(int sock, char *buf, int size)
	{
	 ...
	}

处理方式就是从sock中往buf里面读size个字符，需要注意的就是当遇到*\r,\n,\r\n*的时候全部统一成*\r\n*,并且不再往后读入，返回已读字符的个数。
	
	void cat(int client, FILE *resource)
	{
	...
	}

和linux下熟悉的命令一样，读取一个文件，并且用send函数发送给客户端

	void headers(int client, const char *filename)
	{
	...
	}

这里就是生成返回的header,只有最简单的Content-type和http 200

	void serve_file(int client, const char *filename)
	{
	...
	}

这里就是用到了*cat*,*headers*两个函数，这里注意的是socket queue里面需要把client发送的header读到一个buffer然后丢掉这些数据，然后才能通过socket返回文件的内容。判断的方法就是header最后跟着一个空行.

*not_found()* *unimplemented()*  *cannot_execute()* *bad_request()* 这几个函数就是直接返回固定的内容，直接返回html，分别对应当文件不存在，请求不支持和脚本无法执行的情形。

	void error_die(const char *sc)
	{
	 perror(sc);
	 exit(1);
	}

*error_die* 函数用perror函数打印错误信息

剩下就是两个核心的函数：*accept_request*和 *execute_cgi*
	
	void accept_request(int client)
	{
	 ...
	}
	
accept_request解析header里面的第一行获得request method(get/post)和请求路径，映射到htdoc文件夹下的路径，其中当对应的文件可执行的时候，就调用execute_cgi函数

execute_cgi函数主要是执行服务器端脚本这段程序需要重点关注,CGI需要fork一个新的进程来执行CGI脚本，父子进程间利用管道进行通信，这个我之前也不太懂,不过读了[这个例子](http://stackoverflow.com/a/2784559)之后我大概明白原理，细节还需要再学习。

总的来说感觉这个服务器确实够超轻量级的,但是话说回来让你直接去读一个工业级的源代码，其复杂程度可能会让你完全感到不知所云,所有复杂性的东西都是基于简单的东西不断的扩展和优化得来的.

有时间还是要学习一下linux下的系统编程，下一步打算学习一下[这个系列](http://limpet.net/mbrubeck/2014/08/08/toy-layout-engine-1.html)，正好对浏览器的内核也有一些了解，也许不一定懂最当下的技术，但是懂一些基本的原理也是有好处的.










[bole]:  http://blog.jobbole.com/79023/

