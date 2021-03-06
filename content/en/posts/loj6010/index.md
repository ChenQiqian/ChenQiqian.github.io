---
title: 「网络流 24 题」数字梯形-费用流
urlname: loj6010
date: 2019-03-30 08:47:48
tags:
- 图论
- 网络流
- 费用流
categories:
- OI
- 题解
series:
- 网络流 24 题
libraries:
- mathjax 
##-- toc setting --##
hideToc: false
enableToc: true
enableTocContent: false
---

给定一个由 $n$ 行数字组成的数字梯形如下图所示。梯形的第一行有 $m$ 个数字。从梯形的顶部的 $m$ 个数字开始，在每个数字处可以沿左下或右下方向移动，形成一条从梯形的顶至底的路径。

分别遵守以下规则：
1. 从梯形的顶至底的 $m$ 条路径互不相交；
2. 从梯形的顶至底的 $m$ 条路径仅在数字结点处相交；
3. 从梯形的顶至底的 $m$ 条路径允许在数字结点相交或边相交。

<!--more-->

## 链接

[LOJ6010](https://loj.ac/problem/6010)

## 题解

费用流 3 in 1...

考虑把每个点拆点，然后限制如何满足呢？

1. 我们把所有边和点之间的权值都设置成为 1 
2. 我们把拆点的两个点之间的容量设置成为 inf，其他地方都是 1
3. 所有都是 inf
   
注意原点往最上面一层的每一个点都是容量为 1 的。

## 代码

```cpp
#include <bits/stdc++.h>
#define inf 0x3f3f3f3f
using namespace std;

const int MAXN = 800,MAXM = MAXN*40;

namespace MCMF{//最大费用
  int S,T;
  struct Edge{
    int from,to;
    int cap,flow;
    int cost,nex;
  }edge[MAXM*2];
  int fir[MAXN],ecnt = 2; 
  void addedge(int a,int b,int c,int d){
    edge[ecnt] = (Edge){a,b,c,0, d,fir[a]},fir[a] = ecnt++;
    edge[ecnt] = (Edge){b,a,0,0,-d,fir[b]},fir[b] = ecnt++;
  }
  void clear(){memset(fir,0,sizeof(fir)),ecnt = 2;}
  int dis[MAXN],inq[MAXN];
  bool spfa(){
    memset(dis,0x3f,sizeof(dis));
    static queue<int> q;
    dis[S] = 0;q.push(S);
    while(!q.empty()){
      int x = q.front();q.pop();inq[x] = 0;
      for(int e = fir[x];e;e = edge[e].nex){
        int v = edge[e].to;
        if(edge[e].cap > edge[e].flow && dis[v] > dis[x] + edge[e].cost){
          dis[v] = dis[x] + edge[e].cost;
          if(!inq[v]) q.push(v),inq[v] = 1;
        }
      }
    }
    return dis[T] < dis[0];
  }
  int dfs(int x,int limit = inf){
    if(x == T || limit == 0) return limit;
    int sumf = 0;inq[x] = 1;
    for(int e = fir[x];e;e = edge[e].nex){
      int v = edge[e].to;
      if(!inq[v] && dis[v] == dis[x] + edge[e].cost){
        int f = dfs(v,min(limit,edge[e].cap - edge[e].flow));
        sumf += f,limit -= f;
        edge[e].flow += f, edge[e^1].flow -= f;
        if(limit == 0) break;
      }
    }
    return sumf;
  }
  pair<int,int> solve(int s,int t){
    S = s,T = t;
    int ansf = 0,ansc = 0;
    while(spfa()){
      int f = dfs(s);
      memset(inq,0,sizeof(inq));
      ansf += f,ansc += f * dis[t];
    }
    return make_pair(ansf,ansc);
  }
}


int m,n;
int v[40][40];
int _h(int i,int j){return ((m+(m+i-2))*(i-1)/2)+j;}
int in(int i,int j){return _h(i,j)*2-1;}
int out(int i,int j){return _h(i,j)*2;}

void init(){
  scanf("%d %d",&m,&n);
  for(int i = 1;i<=n;i++)
    for(int j = 1;j<=m+i-1;j++)
      scanf("%d",&v[i][j]);
}

void solve(){
  int S = 2*_h(n,m+n-1)+1,T = S+1;
  // question 1
  MCMF::clear();
  for(int j = 1;j<=m;j++) MCMF::addedge(S,in(1,j),1,0);
  for(int i = 1;i<=n;i++){
    for(int j = 1;j<=m+i-1;j++){
      MCMF::addedge(in(i,j),out(i,j),1,-v[i][j]);
      if(i == n) MCMF::addedge(out(i,j),T,1,0);
      else 
        MCMF::addedge(out(i,j),in(i+1,j),1,0),
        MCMF::addedge(out(i,j),in(i+1,j+1),1,0);
    }
  }
  printf("%d\n",-(MCMF::solve(S,T).second));
  // question 2
  MCMF::clear();
  for(int j = 1;j<=m;j++) MCMF::addedge(S,in(1,j),1,0);
  for(int i = 1;i<=n;i++){
    for(int j = 1;j<=m+i-1;j++){
      MCMF::addedge(in(i,j),out(i,j),m,-v[i][j]);
      if(i == n) MCMF::addedge(out(i,j),T,m,0);
      else 
        MCMF::addedge(out(i,j),in(i+1,j),1,0),
        MCMF::addedge(out(i,j),in(i+1,j+1),1,0);
    }
  }
  printf("%d\n",-(MCMF::solve(S,T).second));
  // question 3
  MCMF::clear();
  for(int j = 1;j<=m;j++) MCMF::addedge(S,in(1,j),1,0);
  for(int i = 1;i<=n;i++){
    for(int j = 1;j<=m+i-1;j++){
      MCMF::addedge(in(i,j),out(i,j),m,-v[i][j]);
      if(i == n) MCMF::addedge(out(i,j),T,m,0);
      else 
        MCMF::addedge(out(i,j),in(i+1,j),m,0),
        MCMF::addedge(out(i,j),in(i+1,j+1),m,0);
    }
  }
  printf("%d\n",-(MCMF::solve(S,T).second));
}

int main(){
  init();
  solve();
  return 0;
}
```
