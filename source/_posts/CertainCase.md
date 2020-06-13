---
title: 經典算法之輸出方案
date: 2020-06-13 17:15:26
tags: [輸出方案]
mathjax: true
categories:
- 算法
---

## 簡要

**一些能夠做到輸出方案的經典算法的方案輸出方法**
<!--more-->
## 前言

在算法競賽學習中，對算法的學習通常囫圇吞棗，只知道如何給一段程序輸入，它能給你什麼輸出，比如最小點覆盖，都知道最小點覆盖等於最大匹配，但如果要知道具体是哪幾個點組成最小點覆盖，就可能束手無策，但其实只要對算法原理有所理解是可以知道的。因而对經典算法的學習應該做到能從基本算法中得到相應的具體方案。

## 經典算法之輸出方案

### 最小樹形圖(Directed_MST)之朱劉算法

這個算法的大致原理是：
* 對每個點找出最小的入邊，假設取用這些邊，有可能有環，暫稱為“最小環”，但可以證明，一个環最小環，在最小樹形圖中，只要用換掉一條邊。
* 如果無環即是最小樹形圖，如果存在點無入邊则無解
* 把所有環找出來，縮為一點，然後把所有入邊，即可換邊的換邊代價作为新的邊權
* 對新圖重複操作，得到結果為止，每次至少縮去一個點， $O(VE)$

這個算法，是先假設取某些邊，如果不是最小樹形圖，再考慮換邊，所以要得到方案，只要每次換邊時記錄下來，最后用了哪些邊也就可以知道。

由於每次縮點是會對點重新編號的，所以要在最後得到原圖的情況，还要在存儲邊时，記錄記真實端點，每次換邊時記綠新父親，最後把根的父親改為-1即可。

```c++
struct Edge //邊的權和顶點
{
    int u, v；
    int ru, rv;/////////////////////////////////記錄的真實端點
    ll w;
} edge[N * N];
int pre[N], id[N], vis[N], pos；
int fa[N];/////////////////////////////////記綠最小樹形圖中點的父親
ll in[N]; //存最小入邊權,pre[v]為邊的起點

ll Directed_MST(int root, int V, int E)
{
    ll ret = 0; //存最小樹形圖總權值
    while (true)
    {
        int i;
        //1.找每個節點的最小入邊
        for (i = 0; i < V; i++)
            in[i] = INF;        //初始化為無穷大
        for (i = 0; i < E; i++) //遍歷每條邊
        {
            int u = edge[i].u;
            int v = edge[i].v;
            if (edge[i].w < in[v] && u != v) //說明顶點v有條權值較小的入邊，記錄之
            {
                fa[edge[i].rv] = edge[i].ru;/////////////////////////換邊時記新父親
                pre[v] = u;        //節點u指向v
                in[v] = edge[i].w; //最小入邊
                if (u == root)     //這個點就是實際的起點
                    pos = i;
            }
        }
        for (i = 0; i < V; i++) //判斷是否存在最小樹形圖
        {
            if (i == root)
                continue;
            if (in[i] == INF)
                return -1; //除了根以外有點没有入邊,則根無法到達它  說明它是獨立的點 一定不能構成樹形圖
        }
        //2.找環
        int cnt = 0; //記錄環數
        memset(id, -1, sizeof(id));
        memset(vis, -1, sizeof(vis));
        in[root] = 0;
        for (i = 0; i < V; i++) //標記每個環
        {
            ret += in[i]; //記綠權值
            int v = i;
            while (vis[v] != i && id[v] == -1 && v != root)
            {
                vis[v] = i;
                v = pre[v];
            }
            if (v != root && id[v] == -1)
            {
                for (int u = pre[v]; u != v; u = pre[u])
                    id[u] = cnt; //標記節點u為第幾個環
                id[v] = cnt++;
            }
        }
        if (cnt == 0)
            break; //無環，break
        for (i = 0; i < V; i++)
            if (id[i] == -1)
                id[i] = cnt++;
        //3.建立新圖   縮點,重新標記
        for (i = 0; i < E; i++)
        {
            int u = edge[i].u;
            int v = edge[i].v;
            edge[i].u = id[u];
            edge[i].v = id[v];
            if (id[u] != id[v])
                edge[i].w -= in[v];
        }
        V = cnt;
        root = id[root];
    }
    return ret;
}
```
