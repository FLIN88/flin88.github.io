---
title: SuffixArray
date: 2019-04-14 19:44:52
mathjax: true
tags: [字符串]
categories:
- ACM
---
 ## 簡要
　**專治字符串疑難雜症**
  <!--more-->

# 算法特質
* 算法複雜度爲 $O(NlogN)$ 可以解決數據範圍在 $1e5$ 級別的問題  

* 可以給出字符字符串所有後綴之間的信息，將所有後綴按字典序排序，並且給出相鄰排名的後綴的最長公共前綴 $(Longest Common Prefix)$,利用這些信息解決很多複雜的，樸素算法需要 $O(N^2)$ 才能解決的問題


# 實例

## POJ-1743 Musical Theme
### 題意
一首個的譜子以一個數字序列給出，求最長的重複出現的片段的長度，如果兩個片段是重複即兩個片段相同，如果一個片段的數值同時加上或減去上同一個值可以得到另一個，可認爲這兩個片段是相同的

### 題解
一個片段的數值同時加上或減去同一個值，片段裏相鄰兩個數值之間的差別不變，所有只要找出差分後的數組裏最長的出現超過兩次（不重疊）的最長子串的長度就可得到答案  
二分答案，在得到的height數組中，把最長公共前綴大於答案 $cur$ 的後綴各自分成一些組，對每一個組裏看每個後綴起始點的最大值和最小值的差是否大於 $cur$ ，如果大於，說明這兩個後綴的前 $cur$ 個字符是相同的，$LCP>cur$ 且這 $cur$ 個字符不重疊。

### 實現
```C++
bool check(int now)
{
	int mx=-INF,mm=INF;
	for(int i=2;i<=n;++i)
		if(height[i]>=now)
		{
			mx=max(mx,max(sa[i],sa[i-1]));
			mm=min(mm,min(sa[i],sa[i-1]));
			if(mx-mm>now)
				return 1;
		}
		else
			mx=-INF,mm=INF;
	return 0;
}
```

## SPOJ-DISUBSTR Distinct Substrings
### 題意
求出一個字符串中本質不同的子串的個數

### 題解
可以先假設所有子串都不同，再利用 $height$ 數組去重，即可得到答案   
如果每一個子串都不同，則存在$\frac{len\times (len+1)}{2}$ 個子串，兩個後綴之間最長公共前綴長度爲 $height[i]$ 從這兩個後綴的起點，長度爲 $1-height[i]$ 的子串都是相同的，如此去重即可得到答案

### 實現
```C++
int ans=len*(len+1)/2;
for(int i=0;i<len;++i)
	ans-=eight[i];
```

## SPOJ-REPEATS Repeats
### 題意
求一個字符串中，求中間出現連續重複的片段的重複次數。  
如 $babb**abaabaabaaba**b$ 的重複次數爲4;

### 題解
枚舉每一個長度，利用後綴數組進行檢查   
設重複的循環節長度爲 $l$ ，在連續的重複子串內部，$s[i]==s[i+l]$，那麼從這兩個點往前和往後一共可以匹配的長度 $L$，就是重複 $k-1$ 次的長度。則 $k=\frac{L}{l}+1$，對每個長度進行枚舉，複雜度爲$O(n)+O(\frac{n}{2})+O(\frac{n}{3})+...+O(1)=O(NlogN))$

### 實現
```C++
for(int i=1;i<=n;++i)//枚舉長度
    {
        for(int j=1;j+i<=n;j+=i)
        {
            int temp=LCM(j,j+i);
            int k=j-(i-temp%i); //temp%i 是第一個不完整的循環節往後的匹配長度
				//這裏是回退讓這個循環節完整
            temp=temp/i+1;
            if(k>0&&LCM(k,k+i)>=i)//檢查回退是否真的得到一個完整循環節
                temp++;
            ans=max(ans,temp);
        }
    }
```


## POJ-3415 Common Substrings

###  題意
給出兩個字符串和一個整數 $k$ ，找出兩個子串位置不同的，長度大於 $k$ 的公共子串的個數

### 題解
對 $height$ 數組按 $k$ 進行分組，對每個組單獨計數，這裏需要單調棧優化才能滿足時間要求，兩個後綴之間的最長公共前綴爲他們在 $height$ 數組上他們之間的最小的 $height$ 的值，那麼對同一個後綴來說，往後走，$LCP$ 是單調的。詳見代碼註釋

### 實現

```C++
//省略求Sufiix
struct D
{
	//已合併到這裏的屬於A、B的後綴的個數,它後面到當前這些後綴的最小值
    ll cnta,cntb,level;
    D(ll _ca,ll _cb,ll _l):cnta(_ca),cntb(_cb),level(_l){}
};
char a[N],b[N];
int k,mid;
ll ans;
vector<pair<int,int> > block;
stack<D> st;
void shrink(int tar)
{
    D _top1=st.top();st.pop();//取出兩個
    D _top2=st.top();st.pop();
    if(_top2.level<tar)//如果靠裏面的那個小於當前的值，就先直接結束本次合併，否則結算那些小於當前值的，回導致最小值改變
    {
        st.push(_top2);
        _top1.level=tar;
        st.push(_top1);
        return ;
    }
    ll _cnt=_top1.cnta*_top2.cntb+_top1.cntb*_top2.cnta;//計數
    ans+=_cnt*(_top2.level-k+1);
    _top2.cnta+=_top1.cnta;//把兩個已結算的合併成一個
    _top2.cntb+=_top1.cntb;
    st.push(_top2);//合併後的放回棧中
}

void cal()
{
    if(int(block.size())<2)//如果這個分組裏只有一個後綴，沒有公共
        return ;
    while(!st.empty())//清空棧
        st.pop();
    for(int i=0;i<block.size();++i)//遍歷這個分組
    {
        pair<int,int> &p=block[i];//當前後綴
        D temp(sa[p.first]<mid,sa[p.first]>mid,p.second);
        if(st.empty()||p.second>=st.top().level)//如果比棧頂小
            st.push(temp);
        else
        {
            temp.level=0x3f3f3f3f;//先改成0x3f
            st.push(temp);//入棧
            while(st.top().level>p.second&&int(st.size())>1)//把大於當前值的塊都合併掉
                shrink(p.second);
            temp=st.top();st.pop();
            temp.level=p.second;//合併完成好把棧頂改回當前的正確值
            st.push(temp);
        }
    }
    while((int)st.size()>1)
        shrink(0);
}

int main()
{
    while(scanf("%d",&k)==1&&k)
    {
        scanf("%s%s",a,b);
        int p=1;
        int lena=strlen(a),lenb=strlen(b);
        for(int i=0;i<lena;++i)
            s[p++]=a[i];
        mid=p;
        s[p++]='*';
        for(int i=0;i<lenb;++i)
            s[p++]=b[i];
        len=lena+lenb+1;
        getsa();
        geth();
        ans=0;
        for(int i=1;i<len;++i)
        {
            if(height[i+1]<k)//這裏說明到來了一個新的組
            {
				//結算此個組
				//pair(x,y) x爲後綴的排名，y爲 x+1名的後綴與它的LCP
                block.push_back(make_pair(i,0x3f3f3f3f));//這裏用0x3f因爲這個分組裏它後面沒有x+1名的後綴了
                cal();
                block.clear();
            }
            else
                block.push_back(make_pair(i,height[i+1]));
        }
        block.push_back(make_pair(len,0x3f3f3f3f));
        cal();
        block.clear();
        printf("%lld\n",ans);
    }
}
```

## SPOJ-PHRASES Relevant Phrases of Annihilation
### 題意
給出 $n$ 個字符串，求在所有串中都出現兩次（不重疊）的穿的最長長度

### 題解
先把所有字符串用各不相同的，並且不會出現在字符串中的字符隔開拼接成一個字符串，求出後綴數組   

二分答案，把 $height$ 按答案分組，檢查組中每個字符串中是否都有兩個以上後綴，並且在每個字符串在組中的後綴的起始點最大值和最小值的差十分大於答案。   
**看字符串在組中的後綴起始點最大值和最小值的差是否大於答案**，是判斷是否存在兩個不重疊的相同子串在字符串中的一個技巧

### 實現


```C++
int belong[N];
int n;
char str[N];
vector<int> v[N];
int mm[15],mx[15];
bool check(int cur)
{
	int cnt=-1;
	for(int i=1;i<=len;++i)//利用vector分組的技巧
	{
		if(height[i]<cur)
			v[++cnt].clear();
		v[cnt].push_back(i);
	}
	for(int i=0;i<=cnt;++i)//遍歷每個組
	{
		if(v[i].size()<2*n)//如果少於2*n個後綴在組中
			continue;
		memset(mm,-1,sizeof(mm));//初始化最值
		memset(mx,-1,sizeof(mx));
		for(int j=0;j<v[i].size();++j)
		{
			int k=belong[sa[v[i][j]]];
			mm[k]=(mm[k]==-1?sa[v[i][j]]:min(mm[k],sa[v[i][j]]));
			mx[k]=(mx[k]==-1?sa[v[i][j]]:max(mx[k],sa[v[i][j]]));
		}
		bool flag=1;
		for(int j=1;j<=n;++j)//檢查每個字符串的最值
		{
			if(mm[j]==-1||mx[j]-mm[j]<cur)
			{
				flag=0;
				break;
			}
		}
		if(flag)
			return 1;
	}
	return 0;
}
```
