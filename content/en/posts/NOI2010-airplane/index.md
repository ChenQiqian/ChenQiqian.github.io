---
title: 「NOI2010」航空管制-拓扑排序
urlname: NOI2010-airplane
date: 2018-08-18 19:45:43
tags:
- 图论
- 拓扑排序
categories: 
- OI
- 题解
series:
- NOI
libraries:
- mathjax 
##-- toc setting --##
hideToc: false
enableToc: true
enableTocContent: false
---


假设目前被延误航班共有 $n$ 个，编号为 $1$ 至 $n$ 。机场只有一条起飞跑道，所有的航班需按某个顺序依次起飞（称这个顺序为起飞序列）。定义一个航班的起飞序号为该航班在起飞序列中的位置，即是第几个起飞的航班。

起飞序列还存在两类限制条件：

+ 第一类（最晚起飞时间限制）：编号为 $i$ 的航班起飞序号不得超过 $k_i$ ;

+ 第二类（相对起飞顺序限制）：存在一些相对起飞顺序限制 $(a, b)$ ，表示航班 $a$ 的起飞时间必须早于航班 $b$ ，即航班 $a$ 的起飞序号必须小于航班 $b$ 的起飞序号。

小 $\text{X}$ 思考的第一个问题是，若给定以上两类限制条件，是否可以计算出一个可行的起飞序列。第二个问题则是，在考虑两类限制条件的情况下，如何求出每个航班在所有可行的起飞序列中的最小起飞序号。

<!--more-->

## 链接

[Luogu P1954](https://www.luogu.org/problemnew/show/P1954)

## 题解

对于这些限制，我们发现可以转换成一个拓扑序的问题。但是我们发现最晚的起飞时间限制比较难达成，而两向的相对关系则是比较轻松的，所以我们考虑到可以把整个时间轴反向，那么就变成了所有飞机都有一个最早起飞的限制，那么就可以加一个虚边，按时间解锁，这个问题就可以解决了。

对于第二问，我们就相当于把这个飞机，反向之后能压多后就压多后，将拓扑排序里面的队列变为优先队列，就可以了。

复杂度是 $O(n^2 \log{n})$，貌似不是正解，BZOJ可过，Luogu 需开 O2 。

## 代码


```cpp
#include <cstdio>
#include <vector>
#include <queue>
#include <cstring>
#define inf 0x3f3f3f3f
#define pii pair<int,int>
using namespace std;

const int MAXN = 2100;

int n,m;
int k[MAXN];
vector<int> edge[MAXN];
vector<int> qqq[MAXN];

int ans[MAXN];
priority_queue<pii> q;

int toposort(int w = 0){
  // printf("w:%d\n",w);
  static int in[MAXN];
  memset(in,0,sizeof(in));
  while(!q.empty()) q.pop();
  for(int x = 1;x<=n;x++){
    for(int i = 0;i<edge[x].size();i++){
      in[edge[x][i]]++;
    }
    in[x]++;
  }
  for(int j = n;j>=1;--j){
    for(int i = 0;i<qqq[j].size();i++){
      int x = qqq[j][i];
      if(--in[x] == 0){
        q.push(make_pair((w==x?-inf:k[x]),x));
      }
    }
    int x = q.top().second;q.pop();
    if(x == w) return j;
    ans[j] = x;
    for(int i = 0;i<edge[x].size();i++){
      int v = edge[x][i];
      if(--in[v] == 0){
        q.push(make_pair((w==v?-inf:k[v]),v));
      }
    }
  }
  return 0;
}

void init(){
  scanf("%d %d",&n,&m);
  for(int i = 1;i<=n;i++){
    scanf("%d",&k[i]);
    qqq[k[i]].push_back(i);
  }
  int a,b;
  for(int i = 1;i<=m;i++){
    scanf("%d %d",&a,&b);
    edge[b].push_back(a);
  }
}

void solve(){
  toposort();
  for(int i = 1;i<=n;i++)
    printf("%d ",ans[i]);
  printf("\n");
  for(int i = 1;i<=n;i++)
    printf("%d ",toposort(i));
  printf("\n");
}

int main(){
  init();
  solve();
  return 0;
}
```

