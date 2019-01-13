---
title: Educational Codeforces Round 58 D
date: 2019-01-13 20:19:41
mathjax: true
tags: [數學,DP,樹]
categories:
- ACM
---
 ## 簡要
　**樹上的數論DP，求多個數的GCD，想出的一下子不敢相信複雜度的算法，敢寫就能過**
  <!--more-->
  ## 題意
　　給出一棵帶點權的樹，求出樹上最長的這樣的一條路徑：路徑上所有節點的 $GCD>1$。節點個數、節點權值的範圍都是 $[1,2e5$]。 
  >　[題目入口](http://codeforces.com/contest/1101/problem/D)

## 題解
一開始感覺在 $2e5$ 的範圍里枚舉每個素數 $p$ 也就是枚舉 $GCD$ ，去求由所有被 $p$ 整除的節點構成是樹的最長路徑的做法，時間複雜度上是通不過的，因為 $[1,2e5]$ 里就已經有接近 $2e4$ 個素數，也就是要跑好多遍樹，就一直在思考有沒有別的辦法。  
但真正的正確做法就真的是這樣，一開始對這個複雜度分析不夠到位。正解為：  
1. 枚舉每一個素數因子,也就是枚舉 $GCD$
2. 把帶有此因子的節點按原來的關係構成一些樹。
3. 在這些樹上找最長路徑
4. 所有的最長路徑中的最大值就是答案  


用 $pn[N]$ 數組, $pn[i]$ 中裝帶有素因子 $i$ 的節點的編號。枚舉 $[1,2e5]$ 中每一個數，每一輪枚舉中，如果 $pn[i]$ 中有節點 $x$ ，就把他們字樹上標記出來：$vis[x]=i$   
就是從每一個節點 $x$ 開始在樹上 $BFS$ ,只往帶有因子 $i$ 的節點前進，把這些節點都做上標記。這些被標記的節點按原來的樹的關係就構成了一些小樹。  
這樣再找出這些小樹上的最長路徑，不斷記錄最大值，答案即可得到。
  ## AC CODE
```c++
#include<bits/stdc++.h>
using namespace std;
const int N=3e5;
int n;
//v[i]節點i的權值，p[i]為數i的最小素因子，vis[i]標記
int v[N],p[N],vis[N];
//mp[i]是樹的關係圖，pn[i]里是含有因子i的節點編號
vector<int> mp[N],pn[N];

void getp()//篩出每個數的最小素因子
{
    for(int i=2;i<N;++i)
    {
        if(!p[i])
        {
            for(int j=i;j<N;j+=i)
            {
                if(!p[j])
                    p[j]=i;
            }
        }
    }
}
pair<int,int> dfs(int np,int nx,int d,int f)//求DFS最深的點，和深度,返回：pair<點，深度>
{
    int mx=nx,md=d;
    pair<int,int> re;
    for(int i=0;i<mp[nx].size();++i)
    {
        int x=mp[nx][i];
        if(vis[x]==np&&x!=f)
        {
            re=dfs(np,x,d+1,nx);
            if(re.second>md)
            {
                md=re.second;
                mx=re.first;
            }
        }
    }
    return make_pair(mx,md);
}
int longest(int np,int nx)//求樹上的最長路徑
{
    int t=dfs(np,nx,1,-1).first;//從任意一點找出最遠點
    return dfs(np,t,1,-1).second;//從找出的最遠點出發，再找最遠點
}
int bfs(int np,int nx)
{
    queue<int> q;
    q.push(nx);
    vis[nx]=np;
    while(!q.empty())//先BFS把帶有因子np的連通節點x標記為：vis[x]=np
    {
        int now=q.front();q.pop();
        for(int i=0;i<mp[now].size();++i)
        {
            int x=mp[now][i];
            if(vis[x]!=np&&!(v[x]%np))
            {
                q.push(x);
                vis[x]=np;
            }
        }
    }
    return longest(np,nx);//找此聯通分量的最長路徑
}
int main()
{
    cin>>n;
    for(int i=1;i<=n;++i)
        scanf("%d",&v[i]);
    int u,w;
    for(int i=1;i<n;++i)
    {
        scanf("%d%d",&u,&w);
        mp[u].push_back(w);
        mp[w].push_back(u);
    }
    getp();//篩一下
    for(int i=1;i<=n;++i)
    {
        int x=v[i];
        while(x!=1)
        {
            int y=p[x];
            pn[y].push_back(i);//每個質數y，記錄下能被y整除的節點i
            while(!(x%y))
                x/=y;
        }
    }
    int ans=0;
    for(int i=2;i<N;++i)//相當於枚舉每個素因子
    {
        if(pn[i].size()>0)//如果這個數是樹上某些節點的因子
        {
            for(int j=0;j<pn[i].size();++j)//對所有含有此因子的節點操作
            {
                int x=pn[i][j];
                if(vis[x]!=i)
                    ans=max(ans,bfs(i,x));//操作
            }
        }
    }
    printf("%d\n",ans);
}

```

## 後記
對複雜度的分析能力有待提高，不要被主觀複雜度判斷所欺騙而不敢下手