# 前缀和 & 差分

## 前缀和

### 简介

前缀和可以简单理解为「数列的前$n$的和」，是一种重要的预处理方式，能大大降低查询的时间复杂度。

### 一维前缀和

#### Code

实在没啥可说的，代码：

```c++
#include <iostream>

using namespace std;

const int N = 1e5 + 10;
int n, m, l, r;
int q[N], s[N];

int main()
{
    cin >> n >> m;
    q[0] = 0, s[0] = 0;
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", &q[i]);
        s[i] = s[i - 1] + q[i];
    }


    while (m--)
    {
        scanf("%d%d", &l, &r);
        printf("%d\n", s[r] - s[l - 1]);
    }

    return 0;
}
```

### 二维前缀和

#### Method

求矩阵的和：$\operatorname{sum}_{i, j} = \operatorname{sum}_{i-1, j}+\operatorname{sum}_{i, j-1}+\operatorname{a}_{i, j} - \operatorname{sum}_{i - 1, j-1}$

求某个子矩阵的和：$\operatorname{sum}_{x 2, y 2}-\operatorname{sum}_{x 1-1, y 2}-\operatorname{sum}_{x 2, y 1-1}+\operatorname{sum}_{x 1-1, y 1-1}$

#### Code

```c++
#include <iostream>

using namespace std;

const int N = 1e3 + 10;
int n, m, k;
int x1, y1;
int x2, y2;
int q[N][N], s[N][N];

int main()
{
    scanf("%d%d%d", &n, &m, &k);
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            scanf("%d", &q[i][j]);

    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            s[i][j] = q[i][j] + s[i - 1][j] + s[i][j - 1] - s[i - 1][j - 1];

    while (k--)
    {
        scanf("%d%d%d%d", &x1, &y1, &x2, &y2);
        printf("%d\n", s[x2][y2] - s[x2][y1 - 1] - s[x1 - 1][y2] + s[x1 - 1][y1 - 1]);
    }
    return 0;
}
```

## 差分

### 简介

差分是一种和前缀和相对的策略，可以当做是求和的逆运算。

这种策略的定义是令$b_{i}=\left\{\begin{array}{ll}
a_{i}-a_{i-1} & i \in[2, n] \\
a_{1} & i=1
\end{array}\right.$

本质上，差分就是前缀和的逆运算。

### 一维差分

#### Method

所谓差分，其实就是前缀和的逆运算
有这么一个数组a: a1, a2, ..., an
定义数组b为:
b1 = a1
b2 = a2 - a1
b3 = a3 - a2
...
那么此时b就是a的差分。

仔细看不难发现 ai = b1 + b2 + ... + bi
所以a也就是b的前缀和。即可以看出前缀和和差分就是逆运算。

##### 那么差分有什么用？

一般解决这样的插入问题：给定一个区间[l, r]，现在要对该区间所有值+C。
正常做法就是o(n)的每一个加一遍
有了差分，我们只需要将(b[l] + c)然后(b[r] - c)即可。

##### 为什么呢？

当b[l] + c，会发生什么？a_l, a_(l+1), ..., a_n所有数全部加C了
因为使用查分还原原数组的时候，做前缀和，会让这个C对l点以后的值都起作用
那么同理，你想要指定这个区间，把(l+1)点-C就可以不再让r点后的值不在受影响了

##### 技巧

差分数组要构造吗？即对a数组o(n)的跑一遍构造b。
其实可以不做，我们可以把a数组想成全是0，那么b也就是0
然后a数组实际每一个值，就是b数组一次插入操作：b[i] + c , b[i + 1] - c。
直接当做插入操作来处理就行了

#### Code

```c++
#include <iostream>

using namespace std;

const int N = 1e5 + 10;
int n, m;
int a[N], d[N];

void difference_insert(int l, int r, int c)
{
    d[l] += c;
    d[r + 1] -= c;
}

int main()
{
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", &a[i]);
        difference_insert(i, i, a[i]);
    }

    int l, r, c;
    while (m--)
    {
        scanf("%d%d%d", &l, &r, &c);
        difference_insert(l, r, c);
    }

    for (int i = 1; i <= n; i++) a[i] = a[i - 1] + d[i];

    for (int i = 1; i <= n; i++) printf("%d ", a[i]);
    printf("\n");

    return 0;
}
```

### 二维差分

#### Code

```c++
//
// 二维差分其实根本不需要想太多，因为查分就是前缀和的逆运算。直接推出来就行。
//
#include <iostream>

using namespace std;

const int N = 1010;
int n, m, q;
int d[N][N], a[N][N];

void insert(int x1, int y1, int x2, int y2, int c)
{
    d[x1][y1] += c;
    d[x2 + 1][y2 + 1] += c;
    d[x1][y2 + 1] -= c;
    d[x2 + 1][y1] -= c;
}

int main()
{
    scanf("%d%d%d", &n, &m, &q);
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
        {
            scanf("%d", &a[i][j]);
            insert(i, j, i, j, a[i][j]);
        }

    int x1, x2, y1, y2, c;
    while(q--)
    {
        scanf("%d%d%d%d%d", &x1, &y1, &x2, &y2, &c);
        insert(x1, y1, x2, y2, c);
    }

    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            a[i][j] = d[i][j] + a[i - 1][j] + a[i][j - 1] - a[i - 1][j - 1];
            printf("%d ", a[i][j]);
        }
        printf("\n");
    }

    return 0;
}
```

