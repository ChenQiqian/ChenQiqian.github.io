---
title: 「SDOI2013」直径-树的直径
urlname: SDOI2013-diameter
date: 2018-05-12 18:04:01
tags:
- 图论
- 树形结构
- 树的直径
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

定义一棵树上最长的路径为树的直径。树的直径可能不唯一。

给定的一棵 $n$ 个结点的树，求其直径的长度，以及有多少条边满足所有的直径都经过该边。

<!--more-->

## 链接

[Luogu P3304](https://www.luogu.org/problemnew/show/P3304)

[BZOJ 3124](https://www.lydsy.com/JudgeOnline/problem.php?id=3124)

## 题解

很有趣的一道题

首先找直径。先从任取点 $t$ 出发，到达最远的一个点 $u$ 。再从 $u$ 出发，到达最远的点$v$，$u$，$v$之间的路径即为树的直径。

这比较显然。

令 $\delta (u,v)$ 为 $u,v$ 两点间路径，其数值即为路径长度。

**引理**：在一棵树中， $x$ 、 $y$ 和 $z$ 是三个不同的结点。当 $x$ 到 $y$ 的最短路与 $y$ 到 $z$ 的最短路不重合时，$x$ 到 $z$ 的最短路就是这两条最短路的拼接。

**定理1**：在一棵树中，以任意结点出发所能到达的最远结点，一定是该树直径的端点之一。

**证明**：假设这条直径是 $\delta (u,v)$ 。分两种情况：

当出发结点 $y$ 在 $\delta(u,v)$ 上时，假设到达的最远结点 $z$ 不是 $u,v$ 中的任一个。这时将 $\delta(y,z)$ 与不与之重合的 $\delta(y,u)$ 拼接（也可以假设不与之重合的是直径的另一个方向），可以得到一条更长的路径，矛盾。

当出发结点 $y$ 不在 $\delta(u,v)$ 上时，分两种情况：

+ 当 $y$ 到达的最远结点 $z$ 横穿 $\delta(u,v)$ 时，记与之相交的结点为 $x$。此时有 $\delta(y,z)=\delta(y,x)+\delta(x,z)$ 。而此时 $\delta(y,z)>\delta(y,v)$ ，故可得 $\delta(x,z)>\delta(x,v)$ 。由 $1$ 的结论可知该假设不成立。

+ 当 $y$ 到达的最远结点 $z$ 与$\delta(u,v)$不相交时，记 $y$ 到 $v$ 的最短路首先与 $\delta(u,v)$ 相交的结点是 $x$。由假设 $\delta(y,z)>\delta(y,x)+\delta(x,v)$ 。而 $\delta(y,z)+\delta(y,x)+\delta(x,u)$ 又可以形成 $\delta(z,u)$ ，而 $\delta(z,u)>\delta(x,u)+\delta(x,v)+2\delta(y,x)=\delta(u,v)+2\delta(y,x)$ 矛盾。

- - -


先求出了直径，我们就发现一件好玩的事情。

**定理2**：对于一个边权为正数的树，其所有的直径必然两两有交点。

**证明**：设树的一条直径为 $\delta (u,v)$ ，任取另一直径为 $\delta (u',v')$ 。其长度设为 $d$ 。

若两直径有公共部分，显然有公共点。

若没有公共部分，则必有一条路径 $\delta (x,y)$ 连接两条直径， $x$ 在 $\delta (u,v)$ 上， $y$ 在 $\delta (u',v')$ 上。

在 $\delta(u,x)$ 和 $\delta(x,v)$ 中，不妨设 $\delta(u,x) \geq \frac{1}{2} \times d$ 。同理设 $\delta(u',y) \geq \frac{1}{2} \times d$ ，又因为 $\delta (x,y) > 0$ ，所以 $\delta (u,u') = \delta(u,x) + \delta(x,y) + \delta(y,u') > d = \delta(u,v)$ ，矛盾。

- - -

我们要求的是有多少个边在在所有的直径上。我们已经求得了一条直径 $\delta(u,v)$ 。

令 $x$ 为在 $\delta(u,v)$ 上离 $u$ 点最远的点，满足存在点 $u'$ ，使得 $\delta(x,u') = \delta(x,u)$ ，且 $u \neq u'$ ，则可得 $\delta(u',v)$ 也是一条直径。

同理 $y$ 为在 $\delta(u,v)$ 上离 $v$ 点最远的点，满足存在点 $v'$ ，使得 $\delta(x,v') = \delta(x,v)$ ，且 $v \neq v'$ ，则可得 $\delta(u,v')$ 也是一条直径。

这两个东西都可以在找出直径之后一边扫直径一边 dfs出来。这个时候我们注意到， $x$ 应当在 $y$ 左侧，且 $x$ 在直径左半部， $y$ 在直径右半部，排列顺序大概是这个样子 $u\leftrightarrow x \leftrightarrow y \leftrightarrow v$ 。很容易看出， $x$ 与 $y$ 之间的部分，就是所有直径的公共边。答案即为 $\delta(x,y)$ 。

时间复杂度大约是一个常数比较大的 $O(n)$ 。

## 代码

懒得写 `bfs` ，于是就比较的慢...

```cpp
#include <cstdio>
#include <cstring>
#include <vector>
#include <cctype>
#include <algorithm>
#define ll long long
using namespace std;

namespace fast_io {//快速输入模板
    inline char read(){
        static const int IN_LEN=1000000;static char buf[IN_LEN],*s,*t;
        return s==t?(((t=(s=buf)+fread(buf,1,IN_LEN,stdin))==s)?-1:*s++) : *s++;
    }
    inline void read(int &x){
        static bool iosig;static char c;
        for (iosig=false,c=read();!isdigit(c);c=read()){
            if(c=='-')iosig=true;if(c==-1)return;
        }
        for(x=0;isdigit(c);c=read())
            x=((x+(x<<2))<<1)+(c^'0');
        if(iosig)x=-x;
    }
}using namespace fast_io;

const int MAXN = 300000;
struct Edge{
    int from,to,len;
};
int n,u=0,v=0,fa[MAXN];
ll dis[MAXN],ans1,ans2;
vector<Edge> edge[MAXN];
void addedge(int a,int b,int c){
    edge[a].push_back((Edge){a,b,c});
    edge[b].push_back((Edge){b,a,c});
}

void init(){
    read(n);
    int a,b,c;
    for(int i = 1;i<=n-1;i++){
        read(a),read(b),read(c);
        addedge(a,b,c);
    }
}


void dfs(int nown,int f){//寻找从nown节点出发的最长路
    fa[nown] = f;
    for(int i = 0;i<edge[nown].size();i++){
        Edge e = edge[nown][i];
        if(e.to == f) continue;
        dis[e.to] = dis[nown] + e.len;
        dfs(e.to,nown);
    }
}

void find(){
    memset(dis,0,sizeof(dis));
    dfs(1,0);
    for(int i = 1;i<=n;i++)//第一次搜到的节点记作直径的一个端点u
        if(dis[i] > dis[u])
            u = i;
    memset(dis,0,sizeof(dis));
    dfs(u,0);
    for(int i = 1;i<=n;i++)//第二次搜到的节点记作直径的另一个端点v
        if(dis[i] > dis[v])
            v = i;
}

bool dfs2(int nown,ll len){//dfs寻找是否从某个节点存在长度为len的路径
    if(len == 0) return true;
    for(int i = 0;i < edge[nown].size();i++){
        Edge e = edge[nown][i];
        if(e.to == fa[nown]) continue;
        if(dfs2(e.to,len - e.len))
            return true;
    }
    return false;
}

void solve(){
    static int nex[MAXN];
    int t = v,tmp = 0;//tmp为直径长度
    while(t!=u){//记录从u到v的路径
        nex[fa[t]] = t;
        t = fa[t];
        tmp++;
    }
    //l代表到右节点最近的满足上文性质的点，r代表到左节点最近的满足上文性质的点
    int l = 0,r = tmp,nowt = 0;
    //循环中dis[t] = d(u,t)
    for(t = u;t!=v;t = nex[t]){
        for(int i = 0;i<edge[t].size();i++){
            Edge e = edge[t][i];
            if(e.to == fa[t] || e.to == nex[t]) continue;
            if(dfs2(e.to,dis[t] - e.len)) 
                l = max(nowt,l);//寻找离u最远的t,满足d(u',t) = d(u,t),得到即为x,名字叫做l
            else if(dfs2(e.to,(dis[v] - dis[t])- e.len)) 
                r = min(r,nowt);//寻找离v最远的t,满足d(t,v') = d(t,v),得到即为y,名字叫做r
        }
        nowt++;
    }
    ans1 = dis[v];//直径长度
    ans2 = r - l;//在这里事实上是求了r和l的位置并求出ans2
}


int main(){
    init();
    find();
    solve();
    printf("%lld\n%lld\n",ans1,ans2);
    return 0;   
}
```


