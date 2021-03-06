---
title: 高斯消元法学习笔记
urlname: gaussian-elimination-notes
date: 2018-06-23 20:43:54
tags:
- 数学
- 高斯消元
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


高斯消元法是线性代数中的一个算法，可用来为线性方程组求解，求出矩阵的秩，以及求出可逆方阵的逆矩阵。当用于一个矩阵时，高斯消元法会产生出一个行梯阵式。

<!--more-->

## 怎么消？

### 一个小小的例子

回想一下你的小学生活吧。

老师给了你一个方程组。

$$
\left\\{  
\begin{array}{rc}
	2x+3y = 7 \\\\
	4x-5y = 3 
\end{array}
\right.  
$$

聪慧如你当然能一眼看出来这个东西的答案是：$x = 2,y = 1$，可是你是怎么看出来的呢？老师告诉过你解二元一次方程组的标准做法：加减消元法。

具体来说，就是用一式乘以某比例之后去减二式，把方程组变成如下的样子：

$$
\left\\{  
\begin{array}{rc}
2x+3y = 7 \\\\
0x-11y = -11  
\end{array}
\right.  
$$

然后再把 $y$ 带回一式就可以得到 $x$ ：

$$
\left\\{  
\begin{array}{rc}
2x+0y = 4 \\\\
0x-11y = -11  
\end{array}
\right.  
$$

从而你知道，$x = 2,y = 1$。

恭喜你，你已经完成了高斯消元。

### 再看上面的例子

如果我们把上面方程组的系数抽出来变成一个行列式，就会如下所示:
$$
\begin{array}{}
\left|\begin{array}{cccc}   
    2 & 3  \\\\
    5 & -4 
\end{array}\right|
\quad
\left|\begin{array}{cccc}   
    7  \\\\
    3 
\end{array}\right| 
\end{array}
$$ 

那么我们消元的过程就会如下所示：

$$
\begin{array}{}
\left|\begin{array}{cccc}   
    2 & 3  \\\\
    0 & -11 
\end{array}\right|
\quad
\left|\begin{array}{cccc}   
    7  \\\\
    -11
\end{array}\right| 
\end{array}
$$ 

然后是

$$
\begin{array}{}
\left|\begin{array}{cccc}   
    2 & 0  \\\\
    0 & -11 
\end{array}\right|
\quad
\left|\begin{array}{cccc}   
    4  \\\\
    -11
\end{array}\right| 
\end{array}
$$ 

注意到，最后我们达成了一个目标：**使整个行列式只有对角线上的部分不为 $0$ ，其他部分均为 $0$ 。**这个条件的达成，让我们可以方便的计算出来这个方程组的解。

这也是我们在接下来设计的算法中需要达到的。

## 高斯-约旦消元法

运用上面提到的思想去解多元一次方程组的算法，叫做高斯-约旦消元法（Gauss-Jordan Elimination）。

它有着以下的优点：

+ 方便理解
+ 不用回代
+ 精度较高

它有着以下的缺点：

+ 运行较慢

### 实现

简单来说，它的运行过程是这个样子的：**每次对于第 $i$ 行，让第 $i$ 列除了第 $i$ 行之外均成为 $0$ ，且不破坏前 $i-1$ 列的该性质。**

具体来说，**每次在处理第 $i$ 行时，将第 $i$ 行整行，乘以恰当比例后与除了第 $i$ 行之外的共 $n-1$ 行相减，使得除了第 $i$ 行之外的 $n-1$ 行的第 $i$ 列均为$0$。**
（如果你对于第 $i$ 行第 $i$ 列的数万一是 $0$ 的情况感到困惑，请你先往下看，并假设这个位置上永远不会是 $0$ ）

正确性的说明：

我们需要证明的，就是我们在循环中处理完第 $i$ 行时，不会破坏前 $i-1$ 行的该性质。注意到我们前 $i-1$ 次操作已经使得第 $i$ 行以后的前 $i-1$ 列均成为了 $0$ ，即为如下所示：(将要处理第4行)

$$
\begin{array}{}
\left|\begin{array}{cccc}   
    a & 0 & 0 & 3 & 2 & 3 & 2 & 3\\\\
    0 & b & 0 & 5 & 2 & 3 & 2 & 3\\\\
    0 & 0 & c & 3 & 2 & 3 & 2 & 3\\\\
	0 & 0 & 0 & d & 2 & 3 & 2 & 3\\\\
    0 & 0 & 0 & 7 & 2 & 3 & 2 & 3\\\\
    0 & 0 & 0 & 5 & 2 & 3 & 2 & 3\\\\
    0 & 0 & 0 & 3 & 2 & 3 & 2 & 3\\\\
	0 & 0 & 0 & 8 & 2 & 3 & 2 & 3\\\\      
\end{array}\right|
\end{array}
$$ 

这个时候我们拿第 $i$ 行无论如何与其他 $n-1$ 行相减，都不会使前 $i-1$ 列的数发生任何改变。这是因为第 $i$ 行的前 $i-1$ 列都是 $0$ 。

### 微小的优化

这个算法主要有两个微小的优化：一个是精度上的优化，一个是时间上的优化。

#### 精度优化

注意到我们在处理第 $i$ 行的时候，在第 $i+1 \rightarrow n$ 行之间的这些行与第 $i$ 行完全是可以互换的。而这个时候我们用第 $i$ 行与其他行相减的时候，我们为了能获得更优秀的精度，往往会选择**在第 $i$ 行到第 $n$ 行中，第 $i$ 列的数的绝对值最大的那一行，与第 $i$ 行交换**，然后再进行后面的操作。

大家都知道，浮点数储存时有不可避免的误差，比如 $2.0000001$ 之类。这个时候如果我们需要对这行乘很多倍，就会导致误差的放大，而除法则不会。

#### 时间优化

注意到事实上我们再处理第 $i$ 行的时候只需要处理第 $i+1$ 列之后的列，所以我们在行之间相减的时候，运用这个技巧大约可以减少一半的复杂度。

### 无解的判定

如果我们发现在处理第 $i$ 行的数的时候，所有能选的第 $i$ 列的数都是 $0$ ，那么这个时候，这个方程就是无解或者没有唯一解的。

## 代码

以[Luogu P3389 【模板】高斯消元法](https://www.luogu.org/problemnew/show/P3389)为例子


```cpp
#include <cstdio>
#include <algorithm>
#include <cmath>
using namespace std;

const int MAXN = 110;
const double eps = 1e-7;

bool gauss(double a[MAXN][MAXN],int n){
    for(int i = 1;i<=n;i++){
        int r = i;
        for(int j = i+1;j<=n;j++)
            if(fabs(a[r][i]) < fabs(a[j][i])) r = j;
        // 寻找a[r][i]使其绝对值最大
        if(r!=i)
            for(int j = 1;j<=n+1;j++) 
                swap(a[r][j],a[i][j]);
        // 交换两列
        if(fabs(a[i][i]) < eps) 
            return false;
        // 如果全部都是0，则无解
        for(int j = 1;j<=n;j++)if(j!=i){
            double t = a[j][i]/a[i][i];
            for(int k = i+1;k<=n+1;k++)
                a[j][k] -= a[i][k] * t;
        }
        //使第i个位置的所有其他数都为0
    }
    for(int i = 1;i<=n;i++)
        a[i][n+1]/=a[i][i];
    return true;
}
int n;
double num[MAXN][MAXN];

void init(){
    scanf("%d",&n);
    for(int i = 1;i<=n;i++){
        for(int j = 1;j<=n+1;j++){
            scanf("%lf",&num[i][j]);
        }
    }
}

void solve(){
    if(gauss(num,n))
        for(int i = 1;i<=n;i++)
            printf("%.2lf\n",num[i][n+1]);
    else
        printf("No Solution\n");
}

int main(){
    init();
    solve();
    return 0;
}
```

