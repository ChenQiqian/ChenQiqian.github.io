---
title: 「SDOI2008」递归数列-矩阵快速幂
urlname: SDOI2008-sequence
date: 2018-11-01 23:50:47
tags:
- 矩阵快速幂
- 数学
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
一个由自然数组成的数列按下式定义：

对于 $i \leq k$ ：$a_i = b_i$

对于 $i > k$  : $a_i = c_1a _ {i-1} + c_2a _ {i-2} + ... + c_ka _ {i-k}$

其中 $b_j$ 和 $c_j$ （ $1 \leq j \leq k$）是给定的自然数。写一个程序，给定自然数 $m \leq n$, 计算 $a_m + a _ {m+1} + a _ {m+2} + ... + a_n$, 并输出它除以给定自然数 $p$ 的余数的值。

对于 100% 的测试数据：

$1 \leq k \leq 15,1 \leq m \leq n \leq 10^{18},0 \le b_1, b_2,... b_k, c_1, c_2,..., c_k \leq 10^9,1 \leq p \leq 10^8$

<!--more-->

## 链接

[Luogu P2461](https://www.luogu.org/problemnew/show/P2461)

## 题解

$$
a_i = \sum  _ {j=1}^{k} c_k a _ {j-k}
$$

构造一个矩阵

$$
M_i
=\begin{bmatrix}
S_i  \\
a_i  \\
a _ {i-1}\\
 \vdots\\
a _ {i-k+2}\\
a _ {i-k+1}\\
\end{bmatrix}
$$

转移矩阵为：

$$
Z
=\begin{bmatrix}
1 & c _ {1} & c _ {2} & \cdots & c _ {k-1}   & c _ {k}  \\
0 & c _ {1} & c _ {2} & \cdots & c _ {k-1}   & c _ {k} \\
0 & 1 & 0  & \cdots & 0 & 0\\
 \vdots & \vdots & \vdots & \ddots & 0   & 0\\
0 & 0 & 0 & 1 & 0 & 0 \\
0 & 0 & 0 & 0 & 1  & 0 \\
\end{bmatrix}
$$

初始的矩阵为

$$
C
=\begin{bmatrix}
S_k\\
b_k \\
b _ {k-1}\\
 \vdots\\
b _ {2}\\
b _ {1} \\
\end{bmatrix}
$$

$$
Z \times C = M _ {k+1}\\
Z \times M_i = M _ {i+1}\\
$$

我们有

$$
Z^{n-k} \times C = M _ {n}
$$

矩阵快速幂即可。

## 代码


```cpp
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <algorithm>
#define ll long long
using namespace std;


const int MAXN = 20;

ll N,M,K,P,S;
ll b[MAXN],c[MAXN];

struct Matrix{
  ll num[MAXN][MAXN];
  Matrix(int op = 0){
    memset(num,0,sizeof(num));
    if(op){for(int i = 1;i<MAXN;i++) num[i][i] = 1;}
  }
  ll* operator [] (const int n){
    return num[n];
  }
};

Matrix mul(Matrix &_x,Matrix &_y){
  Matrix ans;
  for(int i = 1;i<=S;i++){
    for(int j = 1;j<=S;j++){
      for(int k = 1;k<=S;k++){
        ans[i][j] += _x[i][k] * _y[k][j];
      }
      ans[i][j] %= P;
    }
  }
  return ans;
}

Matrix pow(Matrix x,ll k){
  Matrix ans(1);
  for(ll i = k;i;i>>=1,x = mul(x,x)){
    if(i & 1) ans = mul(ans,x);
  }
  return ans;
}

Matrix Z,C;

void init(){
  scanf("%lld",&K);
  S = K+1;
  for(int i = 1;i<=K;i++)
    scanf("%lld",&b[i]);
  for(int i = 1;i<=K;i++)
    scanf("%lld",&c[i]);
  scanf("%lld %lld %lld",&M,&N,&P);
  for(int i = 1;i<=K;i++)
    b[i] %= P,c[i] %= P;
}

ll query(ll x){
  ll ans = 0;
  if(x <= K){
    for(int i = 1;i<=x;i++)
      ans += b[i];
  }
  else{
    Matrix a = pow(Z,x-K);
    for(int i = 1;i<=S;i++)
      ans += C[i][1] * a[1][i]; 
  }
  return ans % P;
}

void build(){
  ll sum = 0;
  for(int i = 1;i<=K;i++){
    C[S-i+1][1] = b[i];
    sum += b[i];
  }
  C[1][1] = sum % P;
  Z[1][1] = 1;
  for(int i = 1;i<=K;i++){
    Z[1][i+1] = c[i];
    Z[2][i+1] = c[i];
  }
  for(int i = 2;i<=K;i++){
    Z[i+1][i] = 1;
  }
}

int main(){
  init();
  build();
  printf("%lld\n",((query(N)-query(M-1))%P+P)%P);
  return 0;
}
```


