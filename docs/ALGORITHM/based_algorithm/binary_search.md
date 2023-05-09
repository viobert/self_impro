# 二分

## 简介

二分查找（英语：binary search），也称折半搜索（英语：half-interval search）、对数搜索（英语：logarithmic search），是用来在一个有序数组中查找某一元素的算法。

## Method

二分的本质并不是单调性，它的本质是存在一个性质可以将区间分成两类，根据题目具体分析

### 二分有两种

1. 整数二分
2. 浮点数二分，浮点数没啥难度，主要就是注意边界的误差值就行（可以看一下模板代码就行）

整数二分边界十分的恶心

### 对于整数，模板有两种：

整数二分，给定一个区间[l, r]。根据某种性质，区间可以分为两部分

1. [l, x]         此区间不满足某种性质
2. [x + 1, r]     此区间满足某种性质

就诞生了两个特殊点，不满足性质的临界点，和满足性质的临界点

我们的目标也就是(x) 或者是 (x + 1)

于是乎，模板的两种就出来了

1. 找x处的值       A点
2. 找x + 1处的值   B点

 * |-----red-----||------green--------|
 * l----------------------------------r
 * 0                AB                   (n-1)

#### 模板一  找不满足性质的临界点   即A点（red的右临界）

 * Steps:

    1. 取 mid = (l + r + 1) / 2    注意模板1要加1

    2. check(mid)判断mid是否不满足条件true为不满足

    3. + if true   [mid, r]     <=>  l = mid   （true即不符合性质 == 在red中，那么寻找右边界，就要向有寻找）包含mid，因为mid不符合性质，不在red中，也就是会在green中，而且我们找的就是green的左端点，故包含。

        + if false  [l, mid - 1] <=>  r = mid - 1

#### 模板二  找满足性质的临界点    即B点（green的左临界）

 * Steps:
    1. 取 mid = (l + r) / 2
    2. check(mid)判断mid是否满足条件 true为满足
    3. + if true   [l, mid]      <=>   r = mid  （true即符合性质 == 在green中，那么寻找左边界，就要向左寻找）
        + if false  [mid + 1， r] <=>   l = mid + 1

### 怎么选择模板呢？

 * 一般拿到问题后，先写check函数，根据这个函数判断一下下一步的区间缩小；
 * 如果是 l = mid，那么是第一种，mid处要加1。mid = (l + r + 1) / 2
 * 如果是 r = mid，那么是第二种，mid处不加1。mid = (l + r) / 2

### 为什么+1？

 * 模版一考虑这种情况：l = r - 1，即该 l 和 r 只差 1
 * 此时如果我们的check判断为True，即满足条件，那么下一个区间一定是 l = mid
 * 如果不+1，此时 mid = (l + r) / 2 = l（整数除法会向下取整）
 * 那区间就还是[l, r]，就会发生死循环。

其实光这样也很难明白，看题

## Code

```c++
bool check(int x) {/* ... */} // 检查x是否满足某种性质


// 区间[l, r]被划分成[l, mid - 1]和[mid, r]时使用：
int bsearch_1(int l, int r)
{
    while (l < r)
    {
        int mid = (l + r + 1) >> 1;
        if (check(mid)) l = mid;
        else r = mid - 1;
    }
    return l;  // 注意答案不是mid，答案就是这个区间的唯一一个数。最后这个区间l==r，只有唯一一个数字了，但是mid还停留在上一次循环
}

// 区间[l, r]被划分成[l, mid]和[mid + 1, r]时使用：
int bsearch_2(int l, int r)
{
    while (l < r) {
        int mid = (l + r) >> 1;
        if (check(mid)) r = mid;    // check()判断mid是否满足性质
        else l = mid + 1;
    }
    return l;  // 注意答案不是mid
}





/*
 * 其次是浮点数二分，浮点数二分非常的简单啊，因为没有边界问题。
 * 最后的条件只要区间长度在一个很小的范围内，我们就可以认为找到了答案，你就可以使用l或者r当做answer了
 */
bool check(double x) {/* ... */} // 检查x是否满足某种性质

double bsearch_3(double l, double r)
{
    // eps 表示精度，取决于题目对精度的要求
    // 一般要比题目要求的精度再低两级，如题目要求保留6位小数，那我们就写1e-8，10^-8

    const double eps = 1e-6;
    while (r - l > eps)
    {
        double mid = (l + r) / 2;
        if (check(mid)) r = mid;
        else l = mid;
    }
    return l;
}
```

## Correlation

### 1. 数的范围

> 给定一个按照升序排列的长度为 n 的整数数组，以及 q 个查询。
>
> 对于每个查询，返回一个元素 k 的起始位置和终止位置（位置从 0 开始计数）。
>
> 如果数组中不存在该元素，则返回 `-1 -1`。

```C++
#include <iostream>

using namespace std;

const int N = 1e6 + 10;

int n, m;
int q[N];

// 模版2的check
bool check_min(int mid_value, int x)
{
    if (mid_value >= x) return true;
    else return false;
}

// 模版1的check
bool check_max(int mid_value, int x)
{
    if (mid_value <= x) return true;
    else return false;
}

int main()
{
    scanf("%d%d", &n, &m);
    for (int i = 0; i < n; i++) scanf("%d", &q[i]);
    while (m--)
    {
        int k;
        scanf("%d", &k);

        int l = 0, r = n - 1, mid;
        while (l < r)
        {
            mid = (l + r) >> 1;
            if (check_min(q[mid], k)) r = mid;
            else l = mid + 1;
        }
        if (q[l] != k)
        {
            cout << "-1 -1\n";
        }
        else
        {
            cout << l << " ";

            l = 0;
            r = n - 1;
            while (l < r)
            {
                mid = (l + r + 1) >> 1;
                if (check_max(q[mid], k)) l = mid;
                else r = mid - 1;
            }
            cout << l << "\n";
        }
    }
    return 0;
}

```

