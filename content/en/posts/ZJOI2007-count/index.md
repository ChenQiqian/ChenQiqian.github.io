---
title: 「ZJOI2007」报表统计-平衡树
urlname: ZJOI2007-count
date: 2018-03-03 18:29:17
tags:
- Treap
- 平衡树
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

有一个长度为 $n$ 的整数序列，并且有以下三种操作：

+ `INSERT i k` ：在原数列的第 $i$ 个数后面添加一个新数 $k$ ；如果原数列的第 $i$ 个数已经添加了若干数，则添加在这些数的最后

+ `MIN GAP`：查询相邻两个数的之间差值（绝对值）的最小值

+ `MIN SORT GAP`：查询所有数中最接近的两个数的差值（绝对值）

<!--more-->

## 链接

[Luogu P1110](https://www.luogu.org/problemnew/show/P1110)

## 题解

一道近乎于裸的 `Treap` ，然而由于我十分蒟蒻而且好久没敲 `Treap` ，调了两个小时才调完。

- - -

这道题我们维护两棵平衡树，一颗 $b$ 记录所有相邻数的差的绝对值，一颗 $b$ 记录所有的数；一个列表，记录每个块的最前面和最后面的数。我们注意到询问三的结果随插入的数不增，所以只需要维护一个最小值 $minn$ 就可以了。

- - -

+ `insert` 操作：首先根据列表内容从 $b$ 里删除对应位置块间两数的差，然后把插入后多出来的相邻元素差，这个数与这一块结尾，下一块最前面的数的差分别插入平衡树 $b$ ，注意 `i == n` 时需要特判；并把插入的数加到 $a$ 里面，根据其与前驱后继的差更新 $minn$ ，注意需要判断一下是否这个数已经在平衡树里面存在。

+ 相邻元素的差值最小值：直接在 $b$ 里求最小值并输出。

+ 排序后的最小差值：直接输出 $minn$ 。

## 代码



```cpp
#include <cstdio>
#include <algorithm>
#include <cstdlib>
#include <cctype>
using namespace std;

namespace fast_io {
    ...//隐去快读模版
}using namespace fast_io;

namespace normal_io{
    inline char read(){
        return getchar();
    }
    inline void read(char *c){
    	scanf("%s",c);
    }
    inline void read(int &x){
        scanf("%d",&x);
    }
    inline void print(int x){
        printf("%d",x);
    }
    inline void print(char x){
        putchar(x);
    }
    inline void flush(){
        return;
    }
}using namespace normal_io;

struct treap{
    struct node{
        int val,p,cnt;
        node* son[2];
    };
    const static int MAXN = 1000000;
    int treapcnt;
    node pool[MAXN],*null,*root;
    treap(){
        treapcnt = 0;
        newnode(null);
        srand(19260817);
        null->val = -0x3f3f3f3f;
        null->p = 2147483647;
        null->cnt = 0;
        root = null;
    }
    void rotate(node *&r,int tmp){
        node *t = r->son[1-tmp];
        r->son[1-tmp] = t->son[tmp];
        t->son[tmp] = r;
        r = t;
    }
    void newnode(node *&r){
        r = &pool[treapcnt++];
        r->son[0] = r->son[1] = null;
    }
    void __insert(node *&r,int v){
        if(r == null){
            newnode(r);
            r->val = v;r->p = rand();r->cnt = 1;
        }
        else{
            if(r->val == v)
                r->cnt++;
            else{
                int tmp = v > r->val;
                __insert(r->son[tmp],v);
                if(r->son[tmp]->p < r->p)
                    rotate(r,1-tmp);
            }
        }
    }
    node *find(node *r,int t){
        while(r->son[t]!=null)
            r = r->son[t];
        return r;
    }
    void __erase(node *&r,int v){
        if(r->val == v){
            if(r->cnt > 1)
                r->cnt--;
            else{
                if(r->son[0] == null&&r->son[1] == null){
                    r = null;return;
                }
                else{
                    int tt = r->son[0]->p > r->son[1]->p;
                    rotate(r,1-tt);
                    __erase(r,v);
                }
            }
        }
        else{
            int tmp = v > r->val;
            __erase(r->son[tmp],v);
        }
    }
    node *nei(int v,int t){
        node* nown = root,*last = null;
        while(nown!=null&&nown->val!=v){
            //printf("2\n");
            int tmp = v > nown->val;
            if(tmp!=t) last = nown;
            nown = nown->son[tmp];
        }
        if(nown->son[t]!=null){
            last = find(nown->son[t],1-t);
        }
        return last;
    }
    bool find(int v){
        node *r = root;
        while(r!=null&&r->val!=v){
            int tmp = v > r->val;
            r = r->son[tmp];
        }
        return r != null;
    }
    inline void __print(node *r,int depth = 0){
    	if(r == null) return;
    	else{
        	__print(r->son[0],depth+1);
        	for(int i = 0;i<depth;i++) putchar(' ');
        	printf("val:%d cnt:%d P:%d son?:%d %d\n",r->val,r->cnt,r->p,r->son[0]!=null,r->son[1]!=null);
        	__print(r->son[1],depth+1);
    	}
    }
    void insert(int v){
        __insert(root,v);
    }
    void erase(int v){
        __erase(root,v);
    }
    void print(){
        __print(root);
    }
};
//以上treap常规模版

treap a,b;
const int MAXN = 1000000;

int head[MAXN],tail[MAXN],minn,n,m;
//a是所有数，b是所有相邻数差值
//head记录此块最前数，tail记录最后数。

void init(){
    minn = 0x3f3f3f3f;
    read(n),read(m);
    static int tmp[MAXN];
    for(int i = 1;i<=n;i++){
        int t;read(t);
        a.insert(t);
        head[i] = tail[i] = tmp[i] = t;
    }
    sort(tmp+1,tmp+n+1);
    for(int i = 2;i<=n;i++){
        //更新初始的两个查询答案
        b.insert(abs(head[i]-head[i-1]));
        minn = min(minn,tmp[i]-tmp[i-1]);
    }
}

void solve(){
    char op[20];int x,y;
    for(int i = 1;i<=m;i++){
        read(op);
        if(op[4] == 'G'){
            print(b.find(b.root,0)->val),print('\n');
            //寻找最小相邻元素差值
        }
        else if(op[4] == 'S'){
            print(minn),print('\n');
            //寻找排序元素差值
        }
        else if(op[4] == 'R'){
            read(x),read(y);
            if(x != n){
                b.erase(abs(head[x+1]-tail[x]));
                b.insert(abs(head[x+1]-y));
            }
            b.insert(abs(tail[x]-y));
            tail[x] = y;
            //更新查询2答案
            if(a.find(y)) minn = 0;
            else{
                int low = a.nei(y,0)->val,up = a.nei(y,1)->val;
                minn = min(minn,min(abs(y-low),abs(up-y)));
            }
            a.insert(y);
            //更新查询3答案
        }
        else if(op[4] =='P'){
            a.print();
            printf("------------------------\n");
            b.print();
            //调试用
        }
        //printf("Finish\n");
    }
    
}

int main(){
    init();
    solve();
    flush();
    return 0;
}

```


