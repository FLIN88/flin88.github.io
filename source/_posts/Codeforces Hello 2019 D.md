---
title: Codeforces Hello 2019 D
date: 2019-01-07 19:52:34
mathjax: true
tags: [數學,DP]
categories:
- ACM
---
 ## 簡要
　**求數學期望，利用積性函數性質進行DP，從數學零基礎認識積性函數, %DYUME**
  <!--more-->
  ## 題意
　　給出兩個數 $n$、$k$ ，$n$ 代表初始數字，k代表進行k次操作。  
  每次操作：n等概率的變成它的因子，包括1和n本身，求k次操作後得到的n的期望值。  
  >　[題目入口](https://codeforces.com/contest/1097/problem/D)

## 題解
　　用公式來描述，定 $f_{k}(x)$ 爲 $x$ 進行 $k$ 次後的期望，定 $m_{x}$ 爲 $x$ 因子個數（包括1和 $x$ 本身，下文同），$c_{i}$ 爲 $x$ 的因子，則所求的就是:  
 <center>$f_k\left ( x \right )= \frac{1}{m_x}\sum_{i=1}^{m_x}f_{k-1}\left ( c_i \right )$  

本題的 $n$ 最大值 $1e15$ ，$k$ 最大值 $1e4$ ，$n$ 的因子可達數百個，若直接按上式逐個因子進行 $k$ 次遞推計算，顯然時間複雜度上不可行。  

其實這是一個積性函數，但數學功底為零的我並不能證明。積性函數就意味著，若 $p_i$、$p_j$ 為互質的數，則有：  

<center>$f_k\left ( x \right )= f_k\left ( p_i \right ) \times f_k\left ( p_j \right )$

於是可以把 $n$ 拆成素因子的冪次之積形式：  

<center>$n=p{_{1}}^{a_1}p{_{2}}^{a_2}p{_{3}}^{a_3}...p{_{m}}^{a_m}$

那麼每個**素因子的冪次**之間是互質的所以可以得到  

<center>$f_k\left ( x \right )= f_k\left ( p{_{1}}^{a_1} \right ) \times f_k\left ( p{_{2}}^{a_2} \right )\times  f_k\left ( p{_{3}}^{a_3} \right ) \times ...\times f_k\left ( p{_{m}}^{a_m} \right )$  

然後單獨計算出單個 $f_k\left ( p{_{i}}^{a_i} \right )$ 再利用積性函數的性質把所有的單個結果乘起來即可得到最終結果，接下來就是如何求出單個的 $f_k\left ( p{_{i}}^{a_i} \right )$ ，方法是DP
### 如何DP
對於每個 $f_k\left ( p{_{i}}^{a_i} \right )$ ，$p{_{i}}^{a_i}$ 的因子顯然也是 $p_i$ 的冪次，定義數組 $dp[i][j]$ 表示 $p{_{i}}^{a_i}$ 進行到第 $j$ 次時，冪次為 $i$ 的概率。$j = 0$時顯然進行 $0$ 次數是本身，所以 $dp[a_i][0]=1$ ，對於第 $j-1$ 次時，冪次為 $i$ 的數，可以在第 $j$ 次時等概率成為 $[0 ，p{_{i}}^{i}]$ 中的任何任何一個 $p_i$ 的冪次，也就是有 $i+1$ 個等概率的結果，所以對於 $k$ 從 $0$ 到 $i$ ，$dp[i][k]+=dp[i-1][j] \times \frac{1}{i+1}$ 這樣DP的轉移方程就出來了，現在就求出了單個 $f_k\left ( p{_{i}}^{a_i} \right )$ 的值，逐個對所有素因子都求一遍，結果乘起來就是結果了。  
  
  到此此題就做完了，以下是AC代碼:  

  ## AC CODE
```c++
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
const ll mod=1e9+7;
ll factor[200],findex[200];//factor[i]、findex[i]爲第ｉ個素因子和其次數
ll dp[100][10005],inv[1000];//dp[i][j]爲進行了j次後指數爲i的概率
ll n,k,cnt;//cnt爲素因子個數
ll ans;
void getinv() //預處理逆元
{
	inv[1]=1;
	for(int i=2;i<1000;i++)
		inv[i]=(mod-mod/i)*inv[mod%i]%mod;
}
ll DP(int now)
{
	memset(dp,0,sizeof(dp));
	dp[findex[now]][0]=1;//一次都沒走，只有一種可能,指數爲findex[now]
	for(int i=1;i<=k;++i)//要進行k次
	{
		for(int j=0;j<=findex[now];++j)//(i-1)次時指數爲j
		{
			if(dp[j][i-1])//如果(i-1)次時指數爲j有概率的話
			{
				for(int l=0;l<=j;++l)//它可以轉移到第i次時指數<=j的狀態
					dp[l][i]=( dp[l][i] + (dp[j][i-1]*inv[j+1]%mod) )%mod;
			}
		}
	}
	ll t=1,ret=0;
	for(int i=0;i<=findex[now];++i)//用概率求期望
	{
		ret=(ret+t*dp[i][k])%mod;
		t=t*factor[now]%mod;
	}
	return ret;
}
int main()
{
	getinv();
	cin>>n>>k;
	for(int i=2;i<=sqrt(n);++i)//先篩出素因子
	{
		if(n%i==0)
		{
			factor[++cnt]=i;
			while(n%i==0)
			{
				n/=i;
				++findex[cnt];//記錄每個素因子的次數
			}
		}
	}
	if(n>1)
	{
		factor[++cnt]=n;
		findex[cnt]=1;
	}
	ans=1;
	for(int i=1;i<=cnt;++i)//每個素因子分別DP
		ans=ans*DP(i)%mod;
	cout<<ans<<endl;
}

```

## 後記
通過本題，零基礎開始對數學有了一點點了解：
>積性函數的性質  
>* $f(1)==1$  
>* $f(c)=f(a)\times f(b)$.........$a,b$互質時(若不互質時為完全積性函數)

以此可以優化一些數學題，類似情形複雜度優化不下來是時可往這個方向思考。