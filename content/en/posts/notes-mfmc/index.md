---
title: 常见最小费用最大流算法学习笔记
urlname: mfmc-notes
date: 2019-03-22 21:20:12
tags:
- 图论
- 网络流
- 费用流
- 模板
categories:
- OI
- 学习笔记
series:
libraries:
- mathjax 
##-- toc setting --##
hideToc: false
enableToc: true
enableTocContent: true
---

众所周知，最小费用最大流向来是一个算法很多的问题，下面总结了几个常用的最小费用最大流算法。

<!--more-->

## 增广路算法（EK算法）

每次都在原图的残余网络上进行一次最短路（ bellmanford 算法或者 spfa 算法）找出一条从原点到汇点的最短路，然后求出这条最短路上的最大可流流量并流满，直到找不出最短路为止。

最广泛使用的费用流算法。

完整代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int inf = 0x3f3f3f3f;

const int MAXN = 510,MAXM = 100000;

struct Edge{
  int from,to;
  int cap,flow;
  int cost,nex;
}edge[MAXM*2];
int fir[MAXN],ecnt = 2;
void addedge(int a,int b,int c,int d){
  // printf("add:%d %d %d %d\n",a,b,c,d);
  edge[ecnt] = (Edge){a,b,c,0,d,fir[a]};
  fir[a] = ecnt++;
  edge[ecnt] = (Edge){b,a,0,0,-d,fir[b]};
  fir[b] = ecnt++;
}

int dis[MAXN],vis[MAXN],minf[MAXN],pree[MAXN];
queue<int> q;

bool spfa(int s,int t){
  while(!q.empty()) q.pop();
  memset(dis,0x3f,sizeof(dis));
  memset(vis,0,sizeof(vis));
  q.push(s);dis[s] = 0,minf[s] = inf;
  while(!q.empty()){
    int nown = q.front();q.pop();
    vis[nown] = 0;
    for(int nowe = fir[nown];nowe;nowe =  edge[nowe].nex){
      Edge & e = edge[nowe];
      if(dis[e.to] > dis[nown] + e.cost && e.cap > e.flow){
        dis[e.to] = dis[nown] + e.cost;
        minf[e.to] = min(minf[nown],e.cap - e.flow);
        pree[e.to] = nowe;
        if(vis[e.to] == 0){
          q.push(e.to);
          vis[e.to] = 1;
        }
      }
    }
  }
  return dis[t] < inf;
}

int min_cost_flow(int s,int t,int k = inf){
  int ans = 0;
  while(spfa(s,t) && k > 0){
    if(dis[t] > 0) break;
    for(int nown = t,nowe = 0;nown != s;nown = edge[nowe].from){
      nowe = pree[nown];
      edge[nowe].flow += minf[t],edge[nowe^1].flow -= minf[t];
    }
    ans += dis[t] * minf[t];
  }
  return ans;
}
int n,m,k;
char s[MAXN];
char t[MAXN];

bool check(int pos,int len){
  if(pos + len - 1 > n) return 0;
  for(int i = 1;i<=len;i++) if(s[pos + i - 1] != t[i]){
    return 0;
  }
  return 1;
}

int main(){
  scanf("%d",&n),scanf("%s",s+1);
  int S = 0,T = n+2;
  scanf("%d",&m);
  for(int i = 1;i<=m;i++){
    int p;
    scanf("%s %d",t+1,&p);
    int len = strlen(t+1);
    for(int i = 1;i<=n;i++){
      if(check(i,len))
        addedge(i,i+len,1,-p);
    }
  }
  scanf("%d",&k);
  for(int i = 1;i<=n;i++){
    addedge(i,i+1,inf,0);
  }
  addedge(S,1,k,0);
  addedge(n+1,T,k,0);
  printf("%d\n",-min_cost_flow(S,T));
  return 0;
}
```

（CF717G）

## 消圈算法

TBD。

## 连续最短路算法（zkw费用流）

TBD。

## 原始对偶算法

### 实现1

每次进行 spfa ，然后在最短路上做dinic多路增广。

完整代码：

```cpp
#include <bits/stdc++.h>
#define inf 0x3f3f3f3f
using namespace std;

const int MAXN = 5100,MAXM = 51000;

namespace MCMF{
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

int n,m,s,t;

int main(){
  // scanf("%d %d %d %d",&n,&m,&s,&t);
  scanf("%d %d",&n,&m);s = 1,t = n;

  for(int i = 1;i<=m;i++){
    int a,b,c,d;
    scanf("%d %d %d %d",&a,&b,&c,&d);
    MCMF::addedge(a,b,c,d);
  } 
  pair<int,int> ans = MCMF::solve(s,t);
  printf("%d %d\n",ans.first,ans.second);
  return 0;
}
```

### 实现2

简单来说，就是我们可以在残量网络上进行一次最短路操作（bellman ford），然后每次去维护一个label，每次扩展完（使用 dinic 仅在符合条件的道路上更改）更新最短路label（使用 dijkstra 算法），然后使用一个松弛操作，代码如下：

```cpp
void reduce(int s,int t){
  for(int e = 2;e <= ecnt;e++) E0.cost += dis[E0.to] - dis[E0.from];
  delta += dis[s];
}
```

然后就可以跑 dijkstra 了。

完整代码：

```cpp
#include <bits/stdc++.h>
#include <bits/extc++.h>
#include <unistd.h>
#define E0 edge[e]
#define E1 edge[e^1]
#define inf 0x3f3f3f3f
using namespace std;

const int MAXN = 410,MAXM = 15010;

struct Edge{
  int from,to,cap,flow,cost,nex;
}edge[MAXM*2];
int fir[MAXN],ecnt = 2;
void addedge(int a,int b,int c,int d){
  edge[ecnt] = (Edge){a,b,c,0, d,fir[a]},fir[a] = ecnt++;
  edge[ecnt] = (Edge){b,a,0,0,-d,fir[b]},fir[b] = ecnt++;
}
struct Node{
  int x,d;
  bool operator < (const Node &_n)const{return d > _n.d;}
  Node(int _x,int _d):x(_x),d(_d){}
};
// #define Node pair<int,int>
// typedef __gnu_pbds::priority_queue<Node, less<Node>, __gnu_pbds::pairing_heap_tag> heap;
// typedef priority_queue< Node ,vector< Node >,greater<Node>> heap;
typedef priority_queue<Node> heap;


int n,m;
int dis[MAXN],inq[MAXN],vis[MAXN],ansf,ansc,delta;

void reduce(int s,int t){
  for(int e = 2;e <= ecnt;e++) E0.cost += dis[E0.to] - dis[E0.from];
  delta += dis[s];
}

bool bellman(int s,int t){// t 为起点
  static queue<int> q;
  memset(dis,0x3f,sizeof(int)*(n+1));while(!q.empty()) q.pop();
  dis[t] = 0,q.push(t);inq[t] = 1;
  while(!q.empty()){
    int x = q.front();q.pop();inq[x] = 0;
    for(int e = fir[x];e;e = edge[e].nex){
      int v = E0.to,c = E1.cap,f = E1.flow,l = E1.cost;
      if(c > f && dis[v] > dis[x] + l){
        dis[v] = dis[x] + l;
        if(!inq[v]) inq[v] = 1,q.push(v);
      }
    }
  }
  return dis[s] < inf;
}

bool dijkstra(int s,int t){
  memset(dis,0x3f,sizeof(int)*(n+1));
  static heap q;
  dis[t] = 0;q.push(Node(t,0));
  while(!q.empty()){
    Node p = q.top();q.pop();int x = p.x;
    if(p.d != dis[x]) continue;
    for(int e = fir[x];e;e = edge[e].nex){
      int v = E0.to,c = E1.cap,f = E1.flow,l = E1.cost;
      if(c > f && dis[v] > dis[x] + l){
        dis[v] = dis[x] + l,q.push(Node(v,dis[v]));
      }
    }
  }
  return dis[s] < inf;
}

int dfs(int x,int t,int limit = inf){
  if(x == t || limit == 0) return limit;
  vis[x] = 1; // differ from dinic
  int sumf = 0;
  for(int &e = cur[x];e;e = edge[e].nex){
    int v = E0.to,c = E0.cap,f = E0.flow,l = E0.cost;
    if(!vis[v] && c > f && l == 0){
      int newf = dfs(v,t,min(limit,c-f));
      sumf += newf,limit -= newf;
      E0.flow += newf,E1.flow -= newf;
      if(limit == 0) break;
    }
  }
  return sumf;
}

void augment(int s,int t){
  int curf = 0;
  while(memset(vis,0,sizeof(int)*(n+1)),(curf = dfs(s,t))){
    ansf += curf,ansc += curf * delta;
  }
}

void primaldual(int s,int t){
  if(!dijkstra(s,t)) return;
  ansf = ansc = delta = 0;
  do{
    reduce(s,t),augment(s,t);
  }while(dijkstra(s,t));
}

int main(){
  scanf("%d %d",&n,&m);
  int S = 1,T = n;
  for(int i = 1;i<=m;i++){
    int a,b,c,d;
    scanf("%d %d %d %d",&a,&b,&c,&d);
    addedge(a,b,c,d);
  }
  primaldual(S,T);
  printf("%d %d\n",ansf,ansc);
  return 0;
}
```
