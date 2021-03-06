---
title: 「SHOI2016」随机序列-线段树
urlname: SHOI2016-sequence
date: 2018-08-18 20:14:36
tags:
- 线段树
- 数据结构
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

你的面前有 $n$ 个数排成一行，分别为 $a_1,a_2,...,a_n$ 。你打算在每相邻的两个 $a_i$c和 $a _ {i+1}$ 间都插入一个加号、减号或者乘号。那么一共有 $3^{n-1}$ 种可能的表达式。

你对所有可能的表达式的值的和非常感兴趣。但这毕竟太简单了，所以你还打算支持一个修改操作，可以修改某个 $a_i$ 的值。

你能够编写一个程序对每个修改都输出修改完之后所有可能表达式的和吗？注意，修改是永久的，也就是说每次修改都是在上一次修改的基础上进行，而不是在最初的表达式上进行。

<!--more-->

## 链接

[Luogu P4340](https://www.luogu.org/problemnew/show/P4340)

## 题解

好题好题。可以运用人类的智慧解决。

可以发现，所有表达式的和里面，有一些加号和减号其实是可以抵消的。

我们重点关注所有表达式第一个不是乘号的位置。

如果我们有这么一个表达式：

$$
a_1 \times a_2 \times ... \times a_i + (a _ {i+1} ... a _ {n})
$$

那么必然有一个表达式：

$$
a_1 \times a_2 \times ... \times a_i - (a _ {i+1} ... a _ {n})
$$

那么他们的和就是：

$$
2 \times a_1 \times a_2 \times ... \times a_i
$$

这样的对数一共有 $3^{n-i-1}$ 对，所以所有在第 $i$ 个数字后出现第一个非乘号的这样的表达式的和是
$$
2 \times 3^{n-i-1} \times a_1 \times a_2 \times ... \times a_i 
$$

所以我们可以推出所有的表达式的和就是：

$$
\sum _ {i = 1}^{n-1} (2 \times 3^{n-i-1} \times a_1 \times a_2 \times ... \times a_i)
$$

如果按照 $i$ 建立一棵线段树，那么我们发现，每次修改的都是某些连续的区间，除去原来的数，然后乘上新的数即可。因为模数是一个素数，所以除去一个数可以用乘上逆元来代替。

建好树之后只需维护一个支持区间乘法，维护区间和的线段树即可。

时间复杂度：$O(q (\log{n} + \log{\text{mod}}))$

## 代码


```cpp
#include <cstdio>
using namespace std;
typedef long long ll;
#define mod 1000000007

const int MAXN = 110000;

ll pow(ll x,ll k){
    ll ans = 1;
    for(ll i = k;i;i>>=1,x = (x*x)%mod) if(i&1) ans = (ans * x)%mod;
    return ans;
}

ll niyuan(ll x){
    return pow(x,mod-2);
}

namespace SegTree{
ll sum[MAXN<<2],lazy[MAXN<<2];
#define lson (nown<<1)
#define rson (nown<<1|1)
#define mid ((l+r)>>1)
void push_up(int nown){
    sum[nown] = (sum[lson] + sum[rson])%mod;
}
void build(int nown,int l,int r,ll *num){
    lazy[nown] = 1;
    if(l == r)
        sum[nown] = num[l];
    else{
        build(lson,l,mid,num);
        build(rson,mid+1,r,num);
        push_up(nown);
    }
}
void addlabel(int nown,ll v){
    (sum[nown] *= v)%=mod;
    (lazy[nown] *= v)%=mod;
}
void push_down(int nown){
    if(lazy[nown]){
        addlabel(lson,lazy[nown]),addlabel(rson,lazy[nown]);
        lazy[nown] = 1;
    }
}
void update(int nown,int l,int r,int ql,int qr,ll v){
    if(ql <= l && r <= qr){
        addlabel(nown,v); 
    }
    else{
        push_down(nown);
        if(ql <= mid)
            update(lson,l,mid,ql,qr,v);
        if(qr >= mid+1)
            update(rson,mid+1,r,ql,qr,v);
        push_up(nown);
    }
}
ll query(){
    return sum[1];
}
}

int n,m;
ll a[MAXN];
ll s[MAXN];
ll tmp[MAXN];

void init(){
    scanf("%d %d",&n,&m);
    for(int i = 1;i<=n;i++)
        scanf("%lld",&a[i]);
}

void build(){
    s[1] = s[2] = 1;
    for(int i = 3;i<=n;i++)
        s[i] = (s[i-1] * 3)%mod;
    for(int i = 2;i<=n;i++)
        (s[i] *= 2)%=mod;
    tmp[n+1] = 1;
    for(int i = 1;i<=n;i++){
        int ttt = n-i+1;
        tmp[ttt] = (tmp[ttt+1] * a[i])%mod;
    }
    for(int i = 1;i<=n;i++)
        (tmp[i] *= s[i]) %= mod;
    SegTree::build(1,1,n,tmp);
}

void solve(){
    ll pos,v;
    for(int i = 1;i<=m;i++){
        scanf("%lld %lld",&pos,&v);
        SegTree::update(1,1,n,1,n-pos+1,(niyuan(a[pos]) * v)%mod);
        a[pos] = v;
        printf("%lld\n",SegTree::query());
    }
}

int main(){
    init();
    build();
    solve();
    return 0;
}
```


