---
title: 「NOI2009」诗人小G-动态规划+决策单调性
urlname: NOI2009-poet
date: 2018-08-24 14:43:31
tags:
- 动态规划
- 决策单调性
- 二分查找
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

小 $\text{G}$ 是一个出色的诗人，经常作诗自娱自乐。但是，他一直被一件事情所困扰，那就是诗的排版问题。

一首诗包含了若干个句子，对于一些连续的短句，可以将它们用空格隔开并放在一行中，注意一行中可以放的句子数目是没有限制的。小 $\text{G}$ 给每首诗定义了一个行标准长度（行的长度为一行中符号的总个数），他希望排版后每行的长度都和行标准长度相差不远。显然排版时，**不应改变原有的句子顺序**，并且小 $\text{G}$ 不允许把一个句子分在两行或者更多的行内。在满足上面两个条件的情况下，小 $\text{G}$ 对于排版中的每行定义了一个不协调度, 为这行的实际长度与行标准长度差值绝对值的 $P$ 次方，而一个排版的不协调度为所有行不协调度的总和。

小 $\text{G}$ 最近又作了几首诗，现在请你对这首诗进行排版，使得排版后的诗尽量协调（即不协调度尽量小），并把排版的结果告诉他。

<!--more-->

## 链接

[Luogu P1912](https://www.luogu.org/problemnew/show/P1912)

## 题解

没看见加粗的话...因而疯狂不会做，然后回家仔细读题...

然后还是不会做。

设 $dp[i]$ 是前 $i$ 句话的不协调度的最小值，那么我们显然有以下的状态转移方程：
$$
dp[i] = \min _ {j=0}^{i-1}(dp[j] + {|sum[i]-sum[j]+(i-j-1)-L|}^p)
$$
打表发现决策单调性。

---

用队列维护一个决策队列，用三元组 $(p,l,r)$ ，代表在 $[l,r]$ 区间中的最有决策点都是 $p$ ，类似：$11122333356 \rightarrow (1,1,3) + (2,4,5) + (3,6,9) + (5,10,10) + (6,11,11)$ ，然后每次从后往前 $pop$ ，直到不能 $pop$ 掉整段之后再去二分看最后一块具体的分界线在哪里，然后在每次查询之前要把  $r < i-1$ 的区间给 $pop$ 掉。

路径的话记录决策点即可。

## 代码 



```cpp
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <algorithm>
#include <cmath>
//#define ll long long
#define ll long double
using namespace std;

const int MAXN = 110000;

ll pow(ll x,int p){
  ll ans = 1;
  for(int i = 1;i<=p;i++) ans *= x;
  return ans;
}

struct Node{
  int p,l,r;
};

int n,l,p;
char s[MAXN][50];
ll sum[MAXN],dp[MAXN];
int last[MAXN];

ll calc(int i,int j){
  return dp[j] + pow(fabs(sum[i] - sum[j] - l - 1),p);
}

int find(int l,int r,int now,int last){
  // 找到在[l,r]区间内符合 now 决策比 last 优的最前面的位置 
  // last,...,last,[now],now
  while(l!=r){
    int mid = (l+r)>>1;
    if(calc(mid,last) > calc(mid,now))
      r = mid;
    else
      l = mid+1;
  }
  return l;
}

void init(){
  scanf("%d %d %d",&n,&l,&p);
  for(int i = 1;i<=n;i++){
    scanf("%s",s[i]);
    sum[i] = strlen(s[i]);
    sum[i] += sum[i-1] + 1;
  }
}

void solve(){
  static Node q[MAXN];
  memset(dp,0,sizeof(dp));
  memset(last,0,sizeof(last));
  int fi=0,la=1;q[0] = (Node){0,1,n};
  for(int i = 1;i<=n;i++){
    if(fi != la && q[fi].r <= i-1) fi++;
    int j = q[fi].p;
    dp[i] = calc(i,j);last[i] = j;
    if(calc(n,q[la-1].p) < calc(n,i)) continue;// 在n处i都不优于q[la-1].p
    while(fi != la && calc(q[la-1].l,q[la-1].p) > calc(q[la-1].l,i)) la--;
    // pop 掉整个尾段的条件：i 点在 q[la-1].l 甚至都是一个更优的决策点
    if(fi==la) q[la++] = (Node){i,i+1,n};
    else{
      Node &t = q[la-1];
      int x = find(t.l,n,i,t.p);
      t.r = x-1;
      q[la++] = (Node){i,x,n};
    }
  }
}

void output(){
  if(dp[n] > 1e18)
    printf("Too hard to arrange\n");
  else{
    static int route[MAXN];
    int cnt = 0,now = n;
    printf("%lld\n",(long long)(dp[n]));
    while(now != 0){
      route[++cnt] = now;
      now = last[now];
    }
    for(int i = 1,t=cnt;i<=n;i++){
      printf("%s",s[i]);
      if(i!=route[t]) putchar(' ');
      else putchar('\n'),t--;
    }
  }
  printf("--------------------\n");
}

int main(){
  int T;
  scanf("%d",&T);
  for(int i = 1;i<=T;i++){
    init();
    solve();
    output();
  }
  return 0;
}
```




