---
title: 「国家集训队」最长双回文子串-回文自动机
urlname: longest-double-palindrome
date: 2019-03-10 12:25:53
tags:
- 字符串
- 回文自动机
categories: 
- OI
- 题解
series:
- 国家集训队
libraries:
- mathjax 
##-- toc setting --##
hideToc: false
enableToc: true
enableTocContent: true
---

输入长度为 $n$ 的串 $S$ ，求 $S$ 的最长双回文子串 $T$,即可将 $T$ 分为两部分$X$ ， $Y$ ， （ $∣X∣,∣Y∣ \geq 1$ ）且 $X$ 和 $Y$ 都是回文串。

<!--more-->

## 链接

[Luogu P4555](https://www.luogu.com.cn/problem/P4555)

## 题解

几乎是模版题。

正反跑一遍回文自动机，找出每个位置往两个方向的最长回文后缀，加起来就是答案。

时间复杂度: $O(n)$ 。

注意 $0$ 和 `a` 的混淆，务必加上 `s[0] = -1` 。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 110000;

struct PAM{
  int s[MAXN],c[MAXN][26],fa[MAXN],len[MAXN],tcnt,last;
  void init(){
    s[0] = -1;
    last = 0,tcnt = 1;
    fa[0] = fa[1] = 1;
    len[0] = 0,len[1] = -1; 
  }
  int getfail(int n,int p){
    while(s[n-len[p]-1] != s[n]) p = fa[p];
    return p;
  }
  int ins(int n,int x){// 返回最长回文后缀长度
    s[n] = x;
    int p = getfail(n,last),&q = c[p][x];
    if(!q){
      int f = c[getfail(n,fa[p])][x];
      q = ++tcnt,len[q] = len[p]+2,fa[q] = f;
    }
    last = q;
    return len[q];
  }
}A,B;

int n;char s[MAXN];
int a[MAXN],b[MAXN];

int main(){
  scanf("%s",s+1);n = strlen(s+1);
  A.init(),B.init();
  for(int i = 1;i<=n;i++) a[i] = A.ins(i,s[i]-'a');//,printf("%d:%d\n",i,a[i]);
  for(int i = n;i>=1;--i) b[i] = B.ins(n-i+1,s[i]-'a');//,printf("%d:%d\n",i,b[i]);
  int ans = 0;
  for(int i = 1;i<=n-1;i++) ans = max(ans,a[i]+b[i+1]);
  printf("%d\n",ans);
  return 0;
}
```