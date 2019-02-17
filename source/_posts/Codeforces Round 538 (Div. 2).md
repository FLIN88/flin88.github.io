---
title: Codeforces Round 538 (Div. 2)
date: 2019-02-17 16:56:34
mathjax: true
tags: [數學,DP]
categories:
- ACM
---
## 簡要
　**C是簡單的數論實用技巧，D是典型的區間DP，E是卡隨機數的交互題**
  <!--more-->
# C. Trailing Loves (or L'oeufs?)
## 題意
　　給出兩個數 $n$、$b$ ，求 $n!$ 用 $b$ 進制表示時末尾有幾個零
  >　[題目入口](http://codeforces.com/contest/1114/problem/C)

## 題解
　　這裡的實質就是問 $n!$ 能整除 $b$ 多少次，這裡就有一個處理 $n!$ 的因子的技巧:
那就是對每個因子的，計算 $[1,n]$ 中有幾個 $b$ 、$b^2$ 、$b^3$ ……的倍數，把所有的個數加起來，就是 $n!$ 里因子 $b$ 的次數

## AC CODE
```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

ll n, b;
vector<ll> fac,num;//fac素因子，num其對應的次數

int main()
{
    cin>>n>>b;
    for(ll i=2;i*i<=b;i++)		//把b分解成素因子並記錄每個素因子的次數
    {
        if(b%i==0)
        {
            ll cnt=0;
            while(b%i==0)
            {
                b/=i;
                cnt++;
            }
            fac.push_back(i);
			num.push_back(cnt);
        }
    }

    if(b>1)		//如果分解不完全
	{
		fac.push_back(b);
		num.push_back(1);
	}

    ll ans=0x3f3f3f3f3f3f3f3f;
    for (int i=0;i<fac.size();++i)//對於每個因子
    {
        ll cnt=0;
        ll t=n;
        while(t)	
        {
            cnt+=t/fac[i];	// t裡面 fac[i] 的倍數的個數
            t/=fac[i];		//除以 fac[i] 之後是 下次是計算 fac[i] 的次數加一
        }
        cnt/=num[i];	//最後該因子的個數，是b裡該因子的個數的幾倍
        ans=min(ans,cnt);	//取最小值就可以得到答案
    }
    if(fac.empty())		//如果沒有因子
        cout<<0<<endl;
    else
        cout<<ans<<endl;
    return 0;
}

```

## 後記
這個處理 $n!$ 的因子的方法還是非常實用的

# D. Flood Fill
## 題意
　　給出一排塗了顏色的東西，每次可以把一串連續的相同顏色的東西都變成另一種顏色，問最少操作幾次可以把整排東西變成相同的顏色
  >　[題目入口](http://codeforces.com/contest/1114/problem/D)

## 題解
　　其實單獨兩種顏色來看，比如 $RB$ ，要麼變成 $RR$ ，要麼變成 $BB$ ，把一排都變成一種顏色，都是從局部的兩種顏色變成一種顏色，且要麼變成跟左邊的同顏色，要麼變成跟右邊的相同顏色，這裡就是典型的區間DP。  
用 $dp[l][r][0]$ 表示從 $l$ 到 $r$ 邊成與左邊同色的最少次數，用 $dp[l][r][1]$ 表示從 $l$ 到 $r$ 邊成與右邊同色的最少次數，那麼轉移方程如下  

<center> $dp[l][r][0]=min( dp[l+1][r][0]+(a[l]!=a[l+1]) , dp[l+1][r][1]+(a[l]!=a[r]) )$  

<center> $dp[l][r][1]=min( dp[l][r-1][0]+(a[r]!=a[l]) , dp[l][r-1][1]+(a[r-1]!=a[r]) )$  

枚舉長度和左端點來進行狀態轉移，就是一個 $O(n^2)$ 的算法

## AC CODE
```c++
#include<bits/stdc++.h>
using namespace std;
const int N = 5050;
int n;
int a[N],dp[N][N][2];

int main()
{
	cin>>n;
	for(int i=1;i<=n;++i)
		cin>>a[i];
	for(int i=2;i<=n;++i)	//枚舉長度
		for(int j=1;j+i-1<=n;++j)//枚舉左端點
		{
			int l=j,r=j+i-1;
			dp[l][r][0]=min(dp[l+1][r][0]+(a[l]!=a[l+1]),dp[l+1][r][1]+(a[l]!=a[r]));
			dp[l][r][1]=min(dp[l][r-1][0]+(a[r]!=a[l]),dp[l][r-1][1]+(a[r-1]!=a[r]));
		}
	cout<<min(dp[1][n][0],dp[1][n][1])<<endl;
}

```

## 後記
典型區間DP方式


# E. Arithmetic Progression
## 題意
這是一道交互題。它有一個等差數列，但打亂了順序，告訴你有幾個數。可以有兩種提問
1. "$> x$" 如果數列中有 $> x$ 的數，它輸出 $1$ ，否則輸出 $0$
2. "$?$ $i$" 第 $i$ 個數的是什麼  

你有60次提問機會，最後要告訴它最小的數，和這個等差數列的公差  

  >　[題目入口](http://codeforces.com/contest/1114/problem/E)

## 題解
這題並沒有什麼特別的難點，直接用第一種提問在32次之內二分出數列的最大數，然後隨機提問第二個問題直到60次，把已知所有數兩兩作差$d_i$，求出所有$d_i$的GCD就是公差，就可算出最小的數了，這裡是一個求公差的小技巧  
注意這裡的隨機，不用 mt19937 是過不了的！

## AC CODE
```c++
#include<bits/stdc++.h>
using namespace std;
int n,f,cnt;
set<int> s;
int main()
{
	cin>>n;
	int l=0,r=1e9,mid;
	while(r-l>1)	//二分求最大值
	{
		mid=(r+l)>>1;
		cout<<"> "<<mid<<endl;
		fflush(stdout);
		cin>>f;
		if(f==-1)
			return 0;
		if(f)
			l=mid;
		else
			r=mid;
		++cnt;
	}
	mt19937 mt;	//不用mt19937 ，用rand過不了
	while(s.size()<n&&cnt<60)	//隨機獲取數列裡的值
	{
		int i=mt()%n+1;
		cout<<"? "<<i<<endl;
		fflush(stdout);
		cin>>f;
		if(f==-1)
			return 0;
		s.insert(f);
		++cnt;
	}

	int g=0;
	for(set<int>::iterator i=s.begin();i!=s.end();++i)
		for(set<int>::iterator j=s.begin();j!=s.end();++j)
			g=__gcd(g,abs(*i-*j));
	cout<<"! "<<r-(n-1)*g<<" "<<g<<endl;
	return 0;
}

```

## 後記
記住求等差數列公差的小技巧，和，mt19937好用