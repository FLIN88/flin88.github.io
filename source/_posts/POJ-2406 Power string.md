---
title: POJ-2406 Power String
date: 2019-01-26 22:22:43
mathjax: true
tags: [字符串]
categories:
- ACM
---
 ## 簡要
　**利用KMP的next數組性質，求字符串的最小循環節，KMP性質運用**
  <!--more-->
  ## 題意
  求一個字符串的最小循環節，問這個循環節循環了幾次，無循環節就是循環了一次
  >　[題目入口](http://poj.org/problem?id=2406)

## 題解
KMP的next數組的性質運用，下面是相關的兩個有用的結論 （ $len$ 為字符串長度）
> KMP關於字符串循環節的結論  

1. 如果字符串存在循環，則循環節的長度為 $L=len-next[len]$  
循環的次數就為 $N=\frac{len}{L}$  

1. 如果不存在循環，那麼至少需要補充 $k$ 個字符構成循環 $k=len-next[len]$  

> [具體的推導過程入口](https://www.cnblogs.com/chenxiwenruo/p/3546457.html)  


於是這道題就直接得到解決了，計算出 $L=len-next[len]$ 看 $L$ 是否被 $len$ 整除，如果不能輸出1，如果能輸出 $\frac{len}{L}$ 即可
  ## AC CODE
```c++
#include<cstdio>
#include<cstring>
using namespace std;
const int N =1e6+7;
char s[N];
int next[N];
int len;
void getnext()
{
	memset(next,0,sizeof(next));
	int j=0,k=-1;
	next[0]=-1;
	while(j<len)
	{
		while(k!=-1&&s[j]!=s[k])
			k=next[k];
		++j;++k;
		next[j]=k;
	}
}

int main()
{
	while(~scanf("%s",s))
	{
		if(s[0]=='.')
			break;
		len=strlen(s);
		getnext();
		int l=len-next[len];
		if(len%l==0)
			printf("%d\n",len/l);
		else
			puts("1");
	}
}

```
