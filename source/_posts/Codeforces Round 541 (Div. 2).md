---
title: Codeforces Round 541 (Div. 2)
date: 2019-02-24 20:07:45
mathjax: true
tags: [圖論,DP,字符串]
categories:
- ACM
---
## 簡要
　**C是簡單思維，D是有向圖最長路，E是思維遞推字符串，F是并查集加鏈錶**
  <!--more-->
# C. Birthday
## 題意
　　給出一個數組，把所有數排成一圈，求如何排使得相鄰的數字的差的最大值最小，也就是波動最小
  >　[題目入口](http://codeforces.com/contest/1131/problem/C)

## 題解
說是思維，其實是亂來，只要把數組排序然後，把奇數編號的數順序輸出，接著再把偶數編號的數倒數輸出，AC

## AC CODE
```c++
#include<bits/stdc++.h>
using namespace std;
const int N = 1e6+8;
int a[N],f[N];
int n;
int main()
{
	cin>>n;
	for(int i=1;i<=n;++i)
		cin>>a[i];
	sort(a+1,a+1+n);
	for(int i=1;i<=n;i+=2)
	{
		cout<<a[i]<<" ";
		f[i]=1;
	}
	for(int i=n;i>0;--i)
	{
		if(!f[i])
			cout<<a[i]<<" ";
	}
	return 0;
}

```

## 後記
開拓思維

# D. Gourmet choice
## 題意
　　有兩個集合，給出任意的各自來自不同集合的兩個元素的大小比較關係，要求給每個元素賦值，如果能滿足，輸出使得元素之間滿足給出的關係，並且使得賦的最大的值最小
  >　[題目入口](http://codeforces.com/contest/1131/problem/D)

## 題解
其實是個圖論題，把相等的元素用并查集縮點，然後進行拓撲排序，如果有環，則無法滿足這關係，如果沒有環，把元素最小的距離設為1，把每條邊的距離當做1，求所有元素的最長路徑距離，得到的距離就是答案

## AC CODE
```c++
#include<bits/stdc++.h>
using namespace std;
const int N = 2000+8;
char c[N][N];
int n,m,cnt;
int f[N],ran[N],in[N],dis[N],ans[N],head[N];
vector<int> L;
queue<int> q;
struct E
{
	int v,next;
}e[N*N];

int add(int u,int v)
{
	in[v]++;
	e[cnt].v=v;
	e[cnt].next=head[u];
	head[u]=cnt++;
}

int fi(int a)
{
	return a==f[a]?a:f[a]=fi(f[a]);
}
void unite(int a,int b)
{
	int x=fi(a),y=fi(b);
	if(x==y)
		return ;
	if(ran[x]>ran[y])
		f[y]=x;
	else
	{
		if(ran[x]==ran[y])
			ran[y]++;
		f[x]=y;
	}
}
bool topo()		//拓撲排序
{
	for(int i=1;i<=n+m;++i)
		if(fi(i)==i&&!in[fi(i)])
		{
			q.push(fi(i));
			dis[fi(i)]=1;
		}
	if(q.empty())
		return 0;
	while(!q.empty())
	{
		int t=q.front();q.pop();
		L.push_back(t);
		for(int i=head[t];~i;i=e[i].next)
		{
			in[e[i].v]--;
			if(!in[e[i].v])
				q.push(e[i].v);
		}
	}
	for(int i=1;i<=n+m;++i)
		if(fi(i)==i&&in[i])
			return 0;
	for(int i=0;i<L.size();++i)	//按照拓撲序遞推出各個元素的最長距離
	{
		int t=fi(L[i]);
		for(int j=head[t];~j;j=e[j].next)
			dis[e[j].v]=max(dis[e[j].v],dis[t]+1);
	}
	return 1;
}

int main()
{
	scanf("%d%d",&n,&m);
	for(int i=1;i<=n+m;++i)
	{
		head[i]=-1;
		f[i]=i;
		ran[i]=0;
		dis[i]=-0x3f3f3f3f;
	}
	for(int i=1;i<=n;++i)
		scanf("%s",c[i]+1);
	for(int i=1;i<=n;++i)
		for(int j=1;j<=m;++j)
			if(c[i][j]=='=')
				unite(i,n+j);		//并查集縮點
	for(int i=1;i<=n;++i)
		for(int j=1;j<=m;++j)
		{
			if(c[i][j]=='<')	//邊從小的指向大的，使得大的距離遠，滿足編號大
				add(fi(i),fi(j+n));
			if(c[i][j]=='>')
				add(fi(j+n),fi(i));
		}
	if(!topo())
	{
		cout<<"No"<<endl;
		return 0;
	}
	cout<<"Yes"<<endl;
	for(int i=1;i<=n+m;++i)		//把縮在一起的點的距離填回去
		dis[i]=dis[fi(i)];
	for(int i=1;i<=n;++i)
		printf("%d ",dis[i]);
	puts("");
	for(int i=n+1;i<=n+m;++i)
		printf("%d ",dis[i]);
	puts("");
	return 0;
}

```

## 後記
有向無環圖最長路徑典型方法

# E. Arithmetic Progression
## 題意
給出了兩個字符串的乘法的定義，求出所有字符串按順序相乘之後的最長連續相同字符段的長度。
  >　[題目入口](http://codeforces.com/contest/1131/problem/E)

## 題解
觀察可得這個乘法具有結合律，也就是說順序并不影響結果，那麼可以從後面往前面乘，那樣容易計算乘之後的狀態轉移，從最後一個字符串開始往前遞推
容易知道最左邊和右邊的字符串永遠是排在最後面的那個字符串，我們只需要維護幾個數據：  
* $ll$ : 當前最左邊連續相同字符的長度
* $lr$ : 當前最右邊連續相同字符的長度
* $m$ : 當前內部最長連續相同字符的長度
* $f$ : 當前狀態是不是全由同一個字符組成，$f=1$ 表示是
考慮以下的情況下的狀態轉移

1. 當前是由同一個字符構成的，即 $f=1$ ，乘另一個字符串時，就看找出那個字符的最長連續相同字符段長度 $nm$
   * 如果這個字符也是由同一個字符構成，且與當前串的那個字符相同時，那乘過去還是都由這個字符構成，且 $m=(nm+1)\times m+nm,ll=lr=m$ 
   * 如果這個字符不是有同一個字符構成，那麼找出裡面當前的那個字符的最大連續長度，把當前的字符串插到那一段里，得到 $m=(nm+1)\times m+nm$，如果這個字符串的最左邊或最右邊的字符也跟當前的字符相同，且左邊的連續長度為 $nl$ ，右邊的為 $nr$ ，那麼可以擴張 $ll$ 和 $lr$ ，$ll=(nl+1)\times m+nl，lr=(nr+1)\times m+nr;$
2. 如果當前由多個字符構成，即 $f=0$ ，乘另一個字符串，有以下的情況
   * 如果 $l==r$ 那麼可以找出字符串中與這個字符相同的地方，把兩個當前串插入它兩邊，得到一段長度為 $ll+rr+1$ 的連續相同字符段
   * 如果 $l!=r$ 只有可能字符串中出現了 $l$($r$) 時，可組成 $ll+1$($lr+1$) 長度的連續相同字符段
   * 只要把 $m$ 一直取最大值就可以了，因為這是 $ll$ 和 $lr$ 并不能擴張，因為當前的串內部已經中斷了，而乘了之後兩邊一定是當前串

這道題的討論有點繁多，這就是難點，當然還有觀察出滿足結合律
## AC CODE
```c++
#include<bits/stdc++.h>
using namespace std;
const int N = 100000+8;
int n;
string s[N];
int cl(string &str)	//最左邊的連續相同字符的長度
{
	int ret=1;
	for(int i=1;i<str.length();++i)
	{
		if(str[i]==str[0])
			ret++;
		else
			break;
	}
	return ret;
}
int cr(string &str)	//最右邊的連續相同字符的長度
{
	int ret=1;
	for(int i=str.length()-2;i>=0;--i)
	{
		if(str[i]==str[str.length()-1])
			ret++;
		else
			break;
	}
	return ret;
}

int cf(string &str,char c)	//求字符串中字符c的最大連續長度
{
	int ret=0,cnt=0;
	for(int i=0;i<str.length();++i)
	{
		if(c==str[i])
			cnt++;
		else
			cnt=0;
		ret=max(ret,cnt);
	}
	return ret;
}

int cc(string &str)	//求字符串中最大連續同字符長度
{
	char c=0;
	int ret=0,cnt=1;
	for(int i=0;i<str.length();++i)
	{
		if(c==str[i])
			cnt++;
		else
			cnt=1;
		ret=max(ret,cnt);
		c=str[i];
	}
	return ret;
}
int main()
{
	char l,r;	//l是最后一個字符串的最左邊的字符，r是最右邊的字符,無論怎麼乘都不會改變
	int ll=1,lr=1,m=0;
	bool f=0;
	cin>>n;
	for(int i=1;i<=n;++i)
		cin>>s[i];
	l=s[n][0];				//先求出當前的狀態
	r=s[n][s[n].length()-1];
	ll=cl(s[n]);
	lr=cr(s[n]);
	m=cc(s[n]);
	if(ll==s[n].length())
		f=1;
	for(int i=n-1;i>0;--i)
	{
		if(f)	//當前由同一個字符構成
		{
			int nm=cf(s[i],l);
			if(nm==s[i].length())	//如果乘的字符也全由這個字符構成
			{
				m=m*(nm+1)+nm;
				ll=lr=m;
			}
			else	//乘的字符由多種字符構成
			{
				f=0;
				int nl=cl(s[i]),nr=cr(s[i]);
				if(s[i][0]==l)
					ll=(nl+1)*m+nl;
				if(s[i][s[i].length()-1]==r)
					lr=(nr+1)*m+nr;
				m=m*(nm+1)+nm;
			}
		}
		else
		{
			if(l==r)	//如果可以把左右兩端的連續段連起來
			{
				for(int j=0;j<s[i].length();++j)
				{
					if(s[i][j]==l)
						m=max(ll+lr+1,m);
				}
			}
			else
			{
				for(int j=0;j<s[i].length();++j)
				{
					if(s[i][j]==l)
						m=max(m,ll+1);
					if(s[i][j]==r)
						m=max(m,lr+1);
				}
			}
		}
	}
	cout<<m<<endl;
	return 0;
}

```

## 後記
多種情況的討論，需要耐心分析


# F. Asya And Kittens
## 題意
知道有一排數字，每個數字間一個隔板，隨後不斷給出兩個數的相鄰關係，給出關係后，它們之間的隔板就消失，成為同一個格子里的數，按照給出的相鄰關係，還原最開始的數字序列
  >　[題目入口](http://codeforces.com/contest/1131/problem/F)

## 題解
只需要用并查集維護元素位置的合併，然後用鏈錶模擬相鄰元素的拼接，最後輸出鏈錶即可
## AC CODE
```c++
#include<bits/stdc++.h>
using namespace std;

const int N=150000+8;
int f[N],ran[N],n;
struct D
{
	int l,r,next;//l和r是這個塊的最左邊的最右邊的元素，next是尾指針
}a[N];
int fi(int x)
{
	return x==f[x]?x:f[x]=fi(f[x]);
}
void unite(int aa,int bb)
{
	int x=fi(aa),y=fi(bb);
	if(x==y)
		return ;
	if(ran[x]>ran[y])	//把y接到x上
	{
		f[y]=x;	
		a[a[y].r].next=a[x].l;	//y的最右邊的尾指針指向x的最左邊
		a[x].l=a[y].l;			//x的最左邊變成y的最左邊那個
	}
	else					//把x接到y上，同理
	{
		if(ran[x]==ran[x])
			ran[y]++;
		f[x]=y;
		a[a[x].r].next=a[y].l;
		a[y].l=a[x].l;
	}
}

int main()
{
	cin>>n;
	for(int i=1;i<=n;++i)
	{
		f[i]=i;
		ran[i]=0;
		a[i].l=a[i].r=i;
		a[i].next=-1;
	}
	int x,y;
	for(int i=1;i<n;++i)
	{
		cin>>x>>y;
		unite(x,y);
	}
	for(int i=1;i<=n;++i)//找出鏈錶最左邊那個元素
		if(fi(i)==i)
			x=a[i].l;		
	while(x!=-1)//輸出整條鏈錶
	{
		cout<<x<<" ";
		x=a[x].next;
	}
	return 0;
}

```
## 後記
注意鏈錶
