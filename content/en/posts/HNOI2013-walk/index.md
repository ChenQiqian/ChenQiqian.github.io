---
title: 「HNOI2013」游走-期望方程
urlname: HNOI2013-walk
date: 2018-07-19 20:07:48
tags:
- 数学
- 期望
- 高斯消元
categories: 
- OI
- 题解
series:
- 各省省选
libraries:
- mathjax 
##-- toc setting --##
hideToc: false
enableToc: true
enableTocContent: false
---

一个无向连通图，顶点从 $1$ 编号到 $N$ ，边从 $1$ 编号到 $M$ 。 小 Z 在该图上进行随机游走，初始时小 Z 在 $1$ 号顶点，每一步小 Z 以相等的概率随机选 择当前顶点的某条边，沿着这条边走到下一个顶点，获得等于这条边的编号的分数。当小 Z 到达 N 号顶点时游走结束，总分为所有获得的分数之和。 现在，请你对这 $M$ 条边进行编号，使得小 Z 获得的总分的期望值最小。

<!--more-->

## 链接

[Luogu P3232](https://www.luogu.org/problemnew/show/P3232)

## 题解

注意到因为图是给定的，所以我们可以通过算出每个点期望经过的次数再推出每个边经过的期望次数。

因为在每个点选定每条边的概率是相同的，所以我们有以下期望方程，设第 $i$ 个点期望经过次数为 $e1_i$ ，度数为 $d_i$ ，第 $i$ 条边期望经过次数为 $e2_i$ ：

$$
e1_i = \sum _ {j} \frac{e1_j}{d_j},\text{(i,j) has a edge}  
$$

那么第 $i$ 条边 $(u,v)$ 的经过次数的期望就是：

$$
e2_i = \frac{e1_u}{d_u} + \frac{e1_v}{d_v}
$$

其中有一些特殊处理，因为开始一定会经过一次 1 节点，可以理解成起点到 1 号节点只有一条边，所以：
$$
e1_1 = 1+\sum _ {j} \frac{e1_j}{d_j},\text{(1,j) has a edge}
$$

而且这个人走到 $n$ 节点后不会再走回来，所以：$e1_n = 0$。

高斯消元即可。

然后排序贪心就可以了。

## 代码


```cpp
#include <cstdio>
#include <cmath>
#include <vector>
#include <algorithm>
using namespace std;

const int MAXN = 600;
const double eps = 1e-8;
int n,m,in[MAXN];
vector<int> edge[MAXN];

struct Edge{
    int from,to;
    double c;
    bool operator < (const Edge &a)const{
        return c > a.c;
    }
}edgea[MAXN*MAXN];

bool gauss(double a[MAXN][MAXN],int n){
    for(int i = 1;i<=n;i++){
        int r = i;
        for(int j = i+1;j<=n;j++)
            if(fabs(a[j][i]) > fabs(a[r][i])) r = j;
        if(r!=i) for(int j = 1;j<=n+1;j++)
            swap(a[r][j],a[i][j]);
        if(fabs(a[i][i]) < eps)
            return false;
        for(int j = 1;j<=n;j++)if(j!=i){
            double t = a[j][i]/a[i][i];
            for(int k = 1;k<=n+1;k++)
                a[j][k] -= a[i][k] * t;
        }
    }
    for(int i = 1;i<=n;i++)
        a[i][n+1]/=a[i][i];
    return true;
}

void init(){
    scanf("%d %d",&n,&m);
    int a,b;
    for(int i = 1;i<=m;i++){
        scanf("%d %d",&a,&b);	
        edge[a].push_back(b);
        edge[b].push_back(a);
        in[a]++,in[b]++;
        edgea[i] = (Edge){a,b,double(0)};
    }
}


void solve(){
    //a_u -> 点u期望次数
    static double a[MAXN][MAXN];
    for(int u = 1;u<n;u++){
        a[u][u] = 1;
        for(int i = 0;i<edge[u].size();i++){
            int v = edge[u][i];
            if(v == n) continue;
            a[u][v] = -1 / double(in[v]);
        }
    }
    a[1][n] = 1;//很重要！对1的特殊处理
    static double c[MAXN];
    gauss(a,n-1);
    for(int i = 1;i<=n-1;i++) c[i] = a[i][n];
    for(int i = 1;i<=m;i++){
        int u = edgea[i].from,v =  edgea[i].to;
        edgea[i].c = c[u]/in[u]+c[v]/in[v];
    }//计算边的期望经过次数
    sort(edgea+1,edgea+m+1);
    double ans = 0;
    for(int i = 1;i<=m;i++) ans += edgea[i].c * i;
    printf("%.3lf\n",ans);
}

int main(){
    init();
    solve();
    return 0;
}
```

