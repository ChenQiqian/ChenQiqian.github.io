---
title: 「NOI2015」品酒大会-后缀数组
urlname: NOI2015-savour
date: 2018-08-03 19:40:39
tags:
- 字符串
- 后缀数组
- 并查集
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


简单版题意：

给定一个长度为 $n$ 的字符串，和一个长度为 $n$ 的数列 $\{a_n\}$ ，求对于 $r$ 从 $0$ 到 $n-1$ ，所有满足 $1 \leq p < q \leq n$ 且 $lcp(p,q) \geq r$ 的数对个数以及满足上述条件的数对中 $a_p \times a_q$ 的最大值。（ $a_i$ 可以为负数）

<!--more-->

- - -

完整版题意：

一年一度的“幻影阁夏日品酒大会”隆重开幕了。大会包含品尝和趣味挑战 两个环节，分别向优胜者颁发“首席品酒家”和“首席猎手”两个奖项，吸引了众多品酒师参加。

在大会的晚餐上，调酒师 $Rainbow$ 调制了 $n$ 杯鸡尾酒。这 $n$ 杯鸡尾酒排成一行，其中第 $n$ 杯酒 $(1 \leq i \leq n)$ 被贴上了一个标签$s_i$，每个标签都是 $26$ 个小写 英文字母之一。设 $str(l, r)$表示第 $l$ 杯酒到第 $r$ 杯酒的 $r - l + 1$个标签顺次连接构成的字符串。若 $str(p, po) = str(q, qo)$，其中 $1 \leq p \leq po \leq n$, $1 \leq q \leq qo \leq n$, $p ≠ q$, $po - p + 1 = qo - q + 1 = r$ ，则称第 $p$ 杯酒与第 $q$ 杯酒是“ $r$ 相似” 的。当然两杯“ $r$ 相似”$(r > 1)$的酒同时也是“ $1$ 相似”、“ $2$ 相似”、……、“ $(r - 1)$ 相似”的。特别地，对于任意的 $1 ≤ p$ , $q ≤ n$ ， $p ≠ q$ ，第 $p$ 杯酒和第 $q$ 杯酒都 是“ $0$ 相似”的。

在品尝环节上，品酒师 $Freda$ 轻松地评定了每一杯酒的美味度，凭借其专业的水准和经验成功夺取了“首席品酒家”的称号，其中第 $i$ 杯酒 ($1 ≤ i ≤ n$) 的 美味度为 $a_i$ 。现在 $Rainbow$ 公布了挑战环节的问题：本次大会调制的鸡尾酒有一个特点，如果把第 $p$ 杯酒与第 $q$ 杯酒调兑在一起，将得到一杯美味度为 $a_p \times a_q$ 的酒。现在请各位品酒师分别对于 $r = 0,1,2, ⋯ , n - 1$ ，统计出有多少种方法可以 选出 $2$ 杯“ $r$ 相似”的酒，并回答选择 $2$ 杯“ $r$ 相似”的酒调兑可以得到的美味度的最大值。

## 链接

[Luogu P2178](https://www.luogu.org/problemnew/show/P2178)

## 题解

注意到这个东西有 LCP ，所以我们可以上后缀数组乱怼。

$O(n^2)$ 的做法是显而易见的，只需要枚举 $p,q$ ，更新LCP位置的值，从大往小再扫一遍就可以了。

事实上问题所求的东西可以转化成：恰好为 $r$ 的最大值，恰好为 $r$ 的数目。

构造出 `height` 数组，我们可以发现，所有 `lcp` 恰好为 $r$ 的数对，必然经过至少一个 `height` 为 $r$ 的位置，而且它们经过的区域的 `height` 应该全都大于等于 $r$ ，随着 $r$ 的减小这个区域是在不断扩大的，事实上就是大于 $r$ 的联通块在不断减少。

用一个并查集维护一下就好。每次连接两个集合 $x,y$ ，都会产生 $siz[x] \times siz[y]$ 对这样的数对。

因为有负数，所以为了获得最大值，这里我们要同时维护最大、最小值。

这道题细节挺多的，有一个地方不能合并，要特殊处理 $0$ 相似，令人窒息。


## 代码


```cpp

#include <cstdio>
#include <cctype>
#include <algorithm>
using namespace std;
#define inf 0x3f3f3f3f
#define ll long long

const int MAXN = 310000;

namespace fast_io{
    //...
}using namespace fast_io;


namespace SA{
int s[MAXN],sa[MAXN],ht[MAXN],rk[MAXN],x[MAXN],y[MAXN],cnt[MAXN];
void get_sa(int n,int m){
    for(int i = 0;i<m;i++) cnt[i] = 0;
    for(int i = 0;i<n;i++) cnt[s[i]]++;
    for(int i = 1;i<m;i++) cnt[i] += cnt[i-1];
    for(int i = n-1;~i;--i) sa[--cnt[s[i]]] = i;
    m = rk[sa[0]] = 0;
    for(int i = 1;i<n;i++) rk[sa[i]] = s[sa[i]] != s[sa[i-1]]?++m:m;
    for(int j = 1;;j<<=1){
        if(++m == n) break;
        for(int i = 0;i<j;i++) y[i] = n-j+i;
        for(int i = 0,k=j;i<n;i++) if(sa[i] >= j) y[k++] = sa[i]-j;
        for(int i = 0;i<n;i++) x[i] = rk[y[i]];
        for(int i = 0;i<m;i++) cnt[i] = 0;
        for(int i = 0;i<n;i++) cnt[x[i]]++;
        for(int i = 1;i<m;i++) cnt[i] += cnt[i-1];
        for(int i = n-1;~i;--i) sa[--cnt[x[i]]] = y[i],y[i] = rk[i];
        m = rk[sa[0]] = 0;
        for(int i = 1;i<n;i++) rk[sa[i]] =(y[sa[i]]!=y[sa[i-1]]||y[sa[i]+j]!=y[sa[i-1]+j])?++m:m;
    }
}
template<typename T>
int mapCharToInt(int n,const T *str){
    int m = *max_element(str,str+n);
    for(int i = 0;i<=m;i++) rk[i] = 0;
    for(int i = 0;i<n;i++) rk[(int)(str[i])] = 1;
    for(int i = 1;i<=m;i++) rk[i] += rk[i-1];
    for(int i = 0;i<n;i++) s[i] = rk[(int)(str[i])]-1;
    return rk[m]; 
}
void getheight(int n){
    for(int i = 0,h = ht[0] = 0;i<n;i++){
        int j = sa[rk[i]-1];
        while(i+h<n&&j+h<n&&s[i+h]==s[j+h]) h++;
        if(ht[rk[i]] = h) h--;
    }
}
void build(int n,char *str){
    int m = mapCharToInt(++n,str);
    get_sa(n,m);
    getheight(n);
}
}

namespace BCJ{
    int f[MAXN],siz[MAXN];
    ll maxn[MAXN],minn[MAXN];
    void init(int n,ll * val){
        for(int i = 0;i<=n;i++){
            f[i] = i,siz[i] = 1;
            maxn[i] = minn[i] = val[i];
        }
    }
    int find(int x){
        return f[x] == x?x:f[x] = find(f[x]);
    }
    bool same(int x,int y){
        return find(x) == find(y);
    }
    ll unite(int x,int y,ll &ans){
        int fx = find(x),fy = find(y);
        if(fx == fy) return 0;
        ans = max(ans,maxn[fx]*maxn[fy]);
        ans = max(ans,minn[fx]*minn[fy]);
        f[fy] = fx;
        minn[fx] = min(minn[fx],minn[fy]);
        maxn[fx] = max(maxn[fx],maxn[fy]);
        ll res = 1LL*siz[fx] * siz[fy];
        siz[fx] += siz[fy];
        return res;
    }
}

bool cmp(int a,int b){
    return SA::ht[a] > SA::ht[b];
}

char str[MAXN];
int n,m;
ll a[MAXN],ans[MAXN],cnt[MAXN];

void init(){
    static ll min1 = inf,min2 = inf,max1 = -inf,max2 = -inf;
    read(n);
    read(str);
    str[n] = 'a'-1;
    for(int i = 1;i<=n;i++){
        read(a[i]);
        if(a[i] < min1) min1=a[i];
        else if(a[i] < min2) min2=a[i];
        if(a[i] > max1) max1=a[i];
        else if(a[i] > max2) max2=a[i];
        ans[i] = -1LL*inf*inf;      
    }
    ans[0] = max(max1*max2,min1*min2);
}

void solve(){
    static int h[MAXN];
    SA::build(n,str);
    BCJ::init(n,a);
    for(int i = 1;i<=n;i++) h[i] = i;
    sort(h+1,h+n+1,cmp);
    for(int i = 1;i<=n;i++){
        int x = h[i],ht = SA::ht[x];
        if(x!=1) cnt[ht] += BCJ::unite(SA::sa[x]+1,SA::sa[x-1]+1,ans[ht]);
    }  
    for(int i = n-2;i>=0;i--){
        cnt[i] += cnt[i+1];
        ans[i] = max(ans[i],ans[i+1]);
    }
    cnt[0] = 1LL*n*(n-1)/2;
    for(int i = 0;i<n;i++){
        if(!cnt[i]) ans[i] = 0;
        print(cnt[i]),print(' '),print(ans[i]),print('\n');
    }
}

signed main(){
    init();
    solve();
    flush();
    return 0;
}
```

