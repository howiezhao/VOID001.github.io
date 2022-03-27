---
title: ZOJ 3707 Calculate Prime S 矩阵快速幂+Fibonacci的性质
---
=============================================



#####  [December 20, 2014](https://web.archive.org/web/20201020201305/https://void-shana.moe/acmalgo/number-theory/zoj-3707-calculate-prime-s-%e7%9f%a9%e9%98%b5%e5%bf%ab%e9%80%9f%e5%b9%82fibonacci%e7%9a%84%e6%80%a7%e8%b4%a8.html "7:56 pm") 
[VOID001](https://web.archive.org/web/20201020201305/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20201020201305/https://void-shana.moe/acmalgo/number-theory/zoj-3707-calculate-prime-s-%e7%9f%a9%e9%98%b5%e5%bf%ab%e9%80%9f%e5%b9%82fibonacci%e7%9a%84%e6%80%a7%e8%b4%a8.html#respond)





Calculate Prime S



| **Time Limit:** 2000MS |  | **Memory Limit:** 262144KB |  | **64bit IO Format:** %lld & %llu |




Description



Define *S[n]* as the number of subsets of {1, 2, …, *n*} that contain no consecutive integers. For each *S[n]*, if for all *i* (1 ≤ *i* < *n*) gcd(*S[i]*, *S[n]*) is 1, we call that *S[n]* as a *Prime S*. Additionally, *S[1]* is also a *Prime S*. For the *Kth* minimum *Prime S*, we’d like to find the minimum *S[n]*which is multiple of *X* and not less than the *Kth* minimum *Prime S*. Please tell us the corresponding (*S[n]* ÷ *X*) mod *M*.





Input



There are multiple test cases. The first line of input is an integer T indicating the number of test cases.


For each test case, there are 3 integers *K* (1 ≤ *K* ≤ 106), *X* (3 ≤ *X* ≤ 100) and *M* (10 ≤ *M* ≤ 106), which are defined in above description.





Output



For each test case, please output the corresponding (*S[n]* ÷ *X*) mod *M*.





Sample Input




```
1
1 3 10
```



culate Prime S  

Sample Output 


```
1
```



解决这个问题需要的知识储备为 ：


1. Fibonacci数列的性质
2. 矩阵快速幂计算Fibonacci
3. 取模运算的性质 (a/b)%c=(a%b*c)/b


[在这里有Fibonacci数列的基本性质](https://web.archive.org/web/20201020201305/http://1.voidword.sinaapp.com/%e8%bd%acfibonacci-%e6%95%b0%e5%88%97%e7%9a%84%e5%9f%ba%e6%9c%ac%e6%80%a7%e8%b4%a8/ "[转]Fibonacci 数列的基本性质") ，   [快速幂的知识在这里有](https://web.archive.org/web/20201020201305/http://1.voidword.sinaapp.com/%e5%bf%ab%e9%80%9f%e5%b9%82%e7%ae%97%e6%b3%95-neu-1391/ "快速幂算法 NEU 1391")，矩阵快速幂的代码在下面 ，理解了快速幂之后很好理解 。


本题首先要**读清题目**，本题涉及的元素较多 ，滤清思路之后 本题要求的 是 不小于 第 K 个 Prime S 的能被X整除的S 序列里的数字 ，而经过我们小数据打表发现 S序列为Fibonacci数列从第三项开始 也就是 2 3 5 8 13 … 而 Prime S 为 Fibonacci数列里的素数 ，由Fibonacci的性质知，Fibonacci中 第n项为Fibonacci素数的条件是 n为质数 当 n>=5时成立  ，所以我们可以通过这个性质 结合素数表 来获得 第 K个Fibonacci素数在Fibonacci数列里的下标  在代码的注释里有具体例子 。然后 我们就可以往后枚举这个下标 用 矩阵快速幂的方法来求第n个Fibonacci数 ，然后找到能被X整除的 ，并进行 S/X %C = （S%XC）/X 的运算把结果输出即可 **由于我的素数生成函数(筛法)把count的初值设置错误 导致开始一直在错， 另外需要注意 p[1] 和 p[2]的值要特殊处理 至于为何前面已经讲过了**


下面是代码



```
/*************************************************************************
    > File Name: C.cpp
    > Author: VOID\_133
    > ################### 
    > Mail: ################### 
    > Created Time: 2014年12月20日 星期六 13时12分59秒
 ************************************************************************/
#include<iostream>
#include<algorithm>
#include<cstdio>
#include<vector>
#include<cstring>
#include<map>
#include<queue>
#include<stack>
#include<string>
#include<cstdlib>
#include<ctime>
#include<set>
using namespace std;

typedef long long LL;

const LL maxn=16000000;
LL p[maxn];
bool prime[maxn];

void genPrime()
{
	prime[0]=0;
	prime[2]=1;
	memset(prime,true,sizeof(prime));
	//int count=0;						//Here is a big mistake!
	int count=1;						
	for(int i=2;i<maxn;i++)
	{
		if(prime[i])
		{
			p[count++]=i;	
			for(int j=i+i;j<maxn;j+=i)
			{
				prime[j]=false;
			}
		}
	}

	p[1]=3;
	p[2]=4;
	//p[k] Store the index of the k th Prime S in Fibonacci Series
	//Example:
	//p[1]=3 means the index of the first Prime S (which is 2) in Fibonacci Series is 3 
	// VERY IMPORTANT!!!
	return ;
}

typedef struct 
{
	LL m[2][2];
}Matrix;

Matrix multi(Matrix a,Matrix b,LL MOD)
{
	Matrix c;
	LL i,j;
	for(int i=0;i<2;i++)
	{
		for(int j=0;j<2;j++)
		{
			c.m[i][j]=0;
			for(int k=0;k<2;k++)
			{
				c.m[i][j]+=a.m[i][k]*b.m[k][j];
				c.m[i][j]%=MOD;						//Use MOD to make the number smaller
			}
		}
	}
	return c;
}

Matrix per={1,0,0,1};
Matrix samp={1,1,1,0};

Matrix matrixQuickPow(LL k,LL MOD)
{
	Matrix ans=per;
	Matrix p=samp;
	while(k)
	{
		if(k&1)
		{
			ans=multi(ans,p,MOD);
			k--;
		}
		p=multi(p,p,MOD);
		k>>=1;
	}
	return ans;
}

int main(void)
{
	LL X,M,K;
	genPrime();
	int T;
	scanf("%d",&T);
	LL mem;
	while(T--)
	{
		scanf("%lld%lld%lld",&K,&X,&M);
		Matrix ans;
		for(LL i=p[K];;i++)
		{
			ans=matrixQuickPow(i-1,X);
			if(ans.m[0][0]%X==0)
			{
				mem=i;
				break;
			}
		}
		ans=matrixQuickPow(mem-1,X*M);
		LL res=ans.m[0][0]/X;
		printf("%lldn",res);
	}
	return 0;
}
```

 






---


[Number Theory](https://web.archive.org/web/20201020201305/https://void-shana.moe/category/acmalgo/number-theory) [C. Linux](https://web.archive.org/web/20201020201305/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20201020201305/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20201020201305/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20201020201305/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20201020201305/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20201020201305/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20201020201305/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20201020201305/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
[转]Fibonacci 数列的基本性质](https://web.archive.org/web/20201020201305/https://void-shana.moe/acmalgo/%e8%bd%acfibonacci-%e6%95%b0%e5%88%97%e7%9a%84%e5%9f%ba%e6%9c%ac%e6%80%a7%e8%b4%a8.html)
[PREVIOUS 
快速幂算法 NEU 1391](https://web.archive.org/web/20201020201305/https://void-shana.moe/acmalgo/%e5%bf%ab%e9%80%9f%e5%b9%82%e7%ae%97%e6%b3%95-neu-1391.html)

            