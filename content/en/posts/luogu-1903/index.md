---
title: 「国家集训队」数颜色-带修改莫队
date: 2018-03-31 12:21:42
tags:
- 数据结构
- 莫队
categories: 
- OI
- 题解
##-- post setting --##
series:
- 国家集训队
libraries:
- mathjax 
##-- toc setting --##
hideToc: false
enableToc: true
enableTocContent: false
---


墨墨购买了一套 $N$ 支彩色画笔（其中有些颜色可能相同）。墨墨会向你发布如下指令：

1. `Q L R` 代表询问你从第 $L$ 支画笔到第 $R$ 支画笔中共有几种不同颜色的画笔。

2. `R P Col` 把第 $P$ 支画笔替换为颜色 $Col$ 。

<!--more-->

## 链接

[Luogu P1903](https://www.luogu.org/problemnew/show/P1903)

## 题解

带修改的莫队裸题。

主要需要注意的就是自加自减时间。

## 代码


```cpp
#include <cstdio>
#include <algorithm>
#include <cmath>
#include <cctype>
#include <vector>
using namespace std;

namespace fast_io {
    ...
}using namespace fast_io;

const int MAXN = 11000,MAX = 1100000;

int n,m,Q;
int col[MAXN],re_col[MAXN],re_pos[MAXN],cnum = 1;
int l = 0,r = 0,x = 0;

int num[MAX],ans = 0;

struct Query{
    int id,ql,qr,qx,ans;
    //运算符重载
    bool operator < (Query b)const{
        if(ql/Q != b.ql/Q)
            return ql/Q < b.ql/Q;
        if(qr/Q != b.qr/Q) 
            return qr/Q < b.qr/Q;
        return qx < b.qx;
    }
};

bool cmp(Query a,Query b){
    return a.id < b.id;
}

vector<Query> query;

void init(){
    read(n),read(m);Q = sqrt(n*2);
    for(int i = 1;i<=n;i++)
        read(col[i]);
    for(int i = 1;i<=m;i++){
        char op[10];int a,b;
        read(op);read(a),read(b);
        if(op[0] == 'Q'){
            Query w;w.ql = a,w.qr = b,w.qx = cnum-1;
            w.id = i;query.push_back(w);
        }
        else if(op[0] == 'R'){
            re_pos[cnum] = a,re_col[cnum] = b;
            cnum++;
        }
    }
    sort(query.begin(),query.end());
}

//加入第pos个数并更新答案
void add(int pos){
    if(num[col[pos]]++ == 0)
        ans++;
}

//删去第pos个数并更新答案
void del(int pos){
    if(--num[col[pos]] == 0)
        ans--;
}

//进行第times次修改
void change(int times){
    if(l<=re_pos[times]&& re_pos[times] <= r){
        if(num[re_col[times]]++ == 0)
            ans++;
        if(--num[col[re_pos[times]]] == 0)
            ans--;
    }
    swap(re_col[times],col[re_pos[times]]);
}

void solve(){
    for(int i = 0;i<query.size();i++){
        //莫队核心转移
        Query w = query[i];
        while(l > w.ql)  add(--l);
        while(r < w.qr)  add(++r);
        while(l < w.ql)  del(l++);
        while(r > w.qr)  del(r--);
        while(x < w.qx)  change(++x);
        while(x > w.qx)  change(x--);
        query[i].ans = ans;
    }
    sort(query.begin(),query.end(),cmp);
    for(int i = 0;i<query.size();i++)
        print(query[i].ans),print('\n');
}

int main(){
    init();
    solve();
    flush();
    return 0;
}
```

