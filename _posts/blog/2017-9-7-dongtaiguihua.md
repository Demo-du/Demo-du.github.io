---
layout: post
title:  青蛙跳台阶问题
categories: Algorithms
description: 
keywords: 
---


# 问题

有一只青蛙，一次既可以跳一级台阶，也可以跳两级台阶。现在有n级台阶，求这只青蛙有多少种跳法

# 思路

这个问题初看很简单，甚至很多C语言课本（比如我本科的那本。。。）上都有一种解法：递归。这个问题实际上就是求斐波那契系数，既然每次可以跳一级或两级，那么，n阶的情况应该是n-1阶和n-2阶的和。以前的课本上一般是这种形式，我用java写了一下：

	int Fibonacci(int n){
	    if(n<=0){
	      return 0;
	    }
	    if(n==1){
	      return 1;
	    }
	    return Fibonacci(n-1)+Fibonacci(n-2);
	}

这种方法初看是没有问题的，并且也确实是可以解出来的。但是，复杂度太大。我们可以发现，中间的Fibonacci（i）被多次重复计算，而实际上，最好是只计算一次。
因此，我们做了如下修改：

        int Fibonacci(int n){
		int []f=new int [n+1]; //f（n）对应n的结果
        f[0]=1;
        f[1]=1;
        for(int i=2;i<=n;i++){
        	f[i]=f[i-1]+f[i-2];
        }
		return f[n];
	}

这种思路复杂度为O（n）;
# 总结
 
这种方法其实用的是动态规划，分解问题，并且保证了不重复计算。
