---
layout: post
date: 2015-02-07 11:38:21
title: stable-marriage
categories: [Programming,cn]
tags: [algorithm,economics]
---

昨天室友拿着一个defered acceptance algorithm,问我以前经济学课上有没有接触过这个东西，我一想没听过，不过他又说了下背景问题，我顿时恍然大悟，因为这个背景问题其实就是大名鼎鼎的stable-marriage问题，以前在学校的时候学过Gale-Shapley algorithm,其实就是换了个名字而已，不过当时也只是简单了解了一下理论，并没有深入的学习过，所以这次又花了一点时间回顾了一下这个问题

# 问题背景
[stable-marriage] [stable-marriage] 问题一般的设定是有n男,n女，每个人有一个对所有异性的偏好排序，那这个问题的目标就是为男女之间进行匹配，匹配的结果有很多种，但我们实际关心的是其中的稳定解。稳定解的定义是最终的组合中，任意两对(A,B),(C,D)中不存在下面的情况：

* 对A而言,D优于B,且对D而言,A优于C
* 对C而言,B优于D,且对B而言,C优于A

这样一个匹配的结果就是一个稳定的解

# 算法

*Gale-Shapley* 算法是针对n男n女完全偏好时候的一种算法，保证可以得到一个稳定解，其具体内容如下：

* 首先每个男生向自己最偏好的女生发出请求，而每个女生会在自己接到的所有请求里面取最好的一个，拒绝其他的

* 在前一轮中被拒绝的男生将在下一轮中对自己还没有发出过请求的女生里面偏好最高的那一个发出请求，女生的策略不变，会选择请求者中偏好最高的一个，也包括前一轮中自己接受过的，如果她更偏好本一轮请求的男生，那么她会拒绝之前的男生而和本轮请求的人进行配对

* 这样，直到剩余的被拒绝的男生为空集时候，问题得到解决，该算法可以被证明在至多n步之后终止

更详细的了解和证明可以参见这个[slide][defered-algo]

这个算法找到的只是其中的一个解，而且可以证明，这样的算法得到的结果是suitor-optimal的，即男生的偏好会优先得到满足，这其中是存在实质上的"性别歧视"的

关于这样的稳定解的个数并没有准确的公式，但大概是一个指数级增长的趋势，在[stackexchange][upper-bound]上看到的答案:

	For an instance with n men and n women, the trivial upper bound is n!, and nothing better is known. For a lower bound, Knuth (1976) gives an infinite family of instances with Ω(2.28n) stable matchings, and Thurber (2002) extends this family to all n.


# R 实现

稳定的解有很多，不同的解表现出来的结果也不尽相同，比如上面的算法，得到的结果就是男生更被偏爱，在上面的算法稍微做一点改动，我的[R代码][another-algo]主要是用之前在网上找到的一段代码做了几处修改得到的：

	# modify original implementation to support
	# 0 in suitor(reviewer)'s preference matrix to support the suitor to remain single, note that a suitor(reviewer)'s preference must end with consecutive zeros if at all
	daa<-function(nMen,nWomen,m.prefs=NULL,w.prefs=NULL){
	  if (is.null(m.prefs)){	# if no prefs given, make them randomly
	    m.prefs <- replicate(n=nMen,sample(seq(from=1,to=nWomen,by=1)))
	    w.prefs <- replicate(n=nWomen,sample(seq(from=1,to=nMen,by=1)))
	  }
	  m.hist    <- rep(0,length=nMen)	# number of proposals made
	  w.hist    <- rep(0,length=nWomen)	# current mate
	  m.singles <- 1:nMen
	  w.singles <- 1:nWomen
	  m.mat <- matrix(data=1:nMen,nrow=nMen,ncol=nWomen,byrow=F)
	    for (iter in 1:nWomen){		# there are as many rounds as maximal preference orders
	      # look at market: all single men
	      # if history not full (been rejected by all women in his prefs)
	      # look at single male's history
	      # propose to next woman on list
	      offers <- NULL
	      for (i in 1:length(m.singles)){
	        m.hist[m.singles[i]] <- min(m.hist[m.singles[i]]+1,nWomen)	# make next proposal according to single i's count
	        offers[i] <- m.prefs[m.hist[m.singles[i]],m.singles[i]]		# offer if single i is the index of the woman corresponding to current round
	      }
	      approached   <- unique(offers)	# index of women who received offers
	      temp.singles <- m.singles
	      m.singles    <- NULL	# reset singles
	      approached=approached[approached!=0]
	      stay.single <- temp.singles[offers==0]
	      for (j in approached){
	        proposers   <- temp.singles[offers==j]
	        #stay.single <- temp.singles[offers==0]		# guys who prefer staying single at current history
	        for (k in 1:length(proposers)){
	          if (w.hist[j]==0){	# if no history and proposer 
	            if(any(w.prefs[ ,j]==proposers[k]))
	            {w.hist[j] <- proposers[k]}						# is somewhere on preference list, accept
	            else {m.singles <- c(m.singles,proposers[k])} # if a man propose,however the targeted woman would rather be single than choose this man,he remains single
	          } else if (!(length(match(w.prefs[w.prefs[ ,j]==proposers[k],j],w.prefs[ ,j])<match(w.prefs[w.prefs[ ,j]==w.hist[j],j],w.prefs[ ,j])))){ # if a women has no preference for a man
	            m.singles <- c(m.singles,w.hist[j])		# if proposer better, fire current guy
	            w.hist[j] <- proposers[k]	# and take proposer on
	          } else {
	            m.singles <- c(m.singles,proposers[k])	# otherwise k stays single
	          }
	        }	
	      }
	      m.singles <- sort(c(m.singles,stay.single))
	      if (length(m.singles)==0){	# if no singles left: stop
	        return(list(m.prefs=m.prefs,w.prefs=w.prefs,iterations=iter,matches=w.hist,singles=m.singles))
	        break
	      }
	      current.match   <- (matrix(rep(w.hist,each=nMen),nrow=nMen,ncol=nWomen)==m.mat)
	      current.singles <- matrix(m.mat %in% m.singles,nrow=nMen)*2
	    }
	  return(list(m.prefs=m.prefs,w.prefs=w.prefs,iterations=iter,matches=w.hist,match.mat=current.match,singles=m.singles))
	}


相比于原来的代码，主要改进了一下，支持偏好矩阵中用0来代表一个人当前面的偏好都得不到满足的时候会选择保持单身，这样经过最多n轮之后，这个人将依旧保持单身，可以尝试几个例子：


	m.prefs=matrix(c(1,2,3,2,3,1,3,1,2),nrow=3)
	daa(3,3,m.prefs,m.prefs)


由于男生的第一偏好没有冲突无论女性的偏好，第一轮所有的男生都会和自己的第一选择配对成功，事实上这样的配对也是稳定的，因为没有一个男生愿意偏离自己的第一选择

所以最后的结果是(1,1),(2,2),(3,3)

那么女生有没有办法在这个规则下影响结果，改变弱势的情况呢，其实是有的，那就是隐藏自己的偏好，只给出自己的第一偏好，剩下的置为0

	w.prefs=matrix(c(2,0,0,3,0,0,1,0,0),nrow=3)
	daa(3,3,m.prefs,w.prefs)


此时得到的结果为：(1,2),(2,3),(3,1)

是不是有些神奇，其实如果接触过博弈论的话就不难理解，在一个博弈中，每个人的策略都是对方策略的函数，有时候会有类似囚徒困境的情形出现，你不一定会去采取某个行动，但是你只要有这个能力，就会影响对方的决策，进而影响最终的均衡，而在这样的均衡下你可能情况更差，反而是如果隐藏自己的一些信息,可能得到的结果会更好，这大概就是我们常说的大智若愚吧

[stable-marriage]: http://en.wikipedia.org/wiki/Stable_marriage_problem
[defered-algo]:http://www.math.harvard.edu/~eriehl/pechakucha.pdf
[upper-bound]:http://cstheory.stackexchange.com/questions/5619/what-is-the-maximum-number-of-stable-marriages-for-an-instance-of-the-stable-mar
[another-algo]:https://gist.github.com/samuel-liyi/1dba2d9b779d15e55e55