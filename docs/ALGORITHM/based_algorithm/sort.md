# 排序算法

## 简介

**排序算法**（英语：Sorting algorithm）是一种将一组特定的数据按某种顺序进行排列的算法。排序算法多种多样，性质也大多不同。

## 稳定性

稳定性是指相等的元素经过排序之后相对顺序是否发生了改变。

拥有稳定性这一特性的算法会让原本有相等键值的纪录维持相对次序，即如果一个排序算法是稳定的，当有两个相等键值的纪录 ![R](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7) 和 ![S](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)，且在原本的列表中 ![R](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7) 出现在 ![S](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7) 之前，在排序过的列表中 ![R](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7) 也将会是在 ![S](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7) 之前。

基数排序、计数排序、插入排序、冒泡排序、归并排序是稳定排序。

选择排序、堆排序、快速排序、希尔排序不是稳定排序。

## 快速排序

快速排序（英语：Quicksort），又称分区交换排序（英语：partition-exchange sort），简称「快排」，是一种被广泛运用的排序算法。

### Methods

Steps:

1. 确定分界点然后取出值：` [l, r]`  左端`q[l]`，右端`q[r]`，中间`q[(l+r)/2]`
2. 调整：使得分界点的左为小，右为大
3. 递归处理左右两侧

有一个简单的暴力，或者说思想的启发:

1. `a[]`  `b[]`
2. ``q[l, r]``上 `<` 分界点值的去a，>分界点值的去b
3. `a` `b`依次放入`q`

时间复杂的：`nlogn`

 * 模版题背即可，别理解过头

 * 比如      

 * ```c++
      quick_sort(q, l, j);
      quick_sort(q, j + 1, r);
      
      // j 改成 i 就不能取关键点为 q[l]; 同理用j就别用q[r]
      ```

### Code

```c++
#include <iostream>

using namespace std;

const int N = 1e6 + 10;
int n;
int q[N];

void quick_sort(int q[], int l, int r)
{
    if (l >= r) return;

    // 取个边界中间不会退化（数字全部一样的情况下取端点会n^2）
    int x = q[(l + r) / 2], i = l - 1, j = r + 1;

    while (i < j)
    {
        do i++; while (q[i] < x);
        do j--; while (q[j] > x);
        if (i < j) swap(q[i], q[j]);
    }

    quick_sort(q, l, j);
    quick_sort(q, j + 1, r);
}

int main()
{
    scanf("%d", &n);
    for (int i = 0; i < n; i++) scanf("%d", &q[i]);

    quick_sort(q, 0, n - 1);

    for (int i = 0; i < n; i++) printf("%d ", q[i]);

    return 0;
}
```



## 归并排序

归并排序（[merge sort](https://en.wikipedia.org/wiki/Merge_sort)）是高效的基于比较的稳定排序算法。

### Methods

Steps:

1. 确定分界点： [l, r]  左端q[l]，右端q[r]，中间q[(l+r)/2]
2. 归并排序两侧
3. 将有序的两列进行符合一定规则的合并（维护两个指针向后移动）

 * 时间复杂度：`nlogn`
 * 模版题背即可，别理解过头

### Code

```c++
#include <iostream>

using namespace std;

const int N = 1e6 + 10;
int n;
int q[N], temp[N];

void merge_sort(int q[], int l, int r)
{
    if (l >= r) return;

    int mid = (l + r) >> 1;

    merge_sort(q, l, mid);
    merge_sort(q, mid + 1, r);

    int i = l, j = mid + 1;
    int k = 0;

    while (i <= mid && j <= r) {
        if (q[i] > q[j]) temp[k++] = q[j++];
        else temp[k++] = q[i++];
    }
    while (i <= mid) temp[k++] = q[i++];
    while (j <= r) temp[k++] = q[j++];

    for (int i = 0, j = l; j <= r; i++, j++) q[j] = temp[i];

}

int main()
{
    scanf("%d", &n);
    for (int i = 0; i < n; i++) scanf("%d", &q[i]);

    merge_sort(q, 0, n - 1);

    for (int i = 0; i < n; i++) printf("%d ", q[i]);

    return 0;
}
```

### Correlation

#### 1. 求逆序对

> 给定一个长度为 n 的整数数列，请你计算数列中的逆序对的数量。
>
> 逆序对的定义如下：对于数列的第 i 个和第 j 个元素，如果满足` i<j` 且 `a[i]>a[j]`，则其为一个逆序对；否则不是。

```c++
/*
 * 求逆序对的过程本质上使用分治的思想
 * 任意一个区间[l, r]，将它一分为二。它的逆序对，无非有三类：
 * 1. 左区间中的一对
 * 2. 右区间中的一对
 * 3. 左区间中较大的一个和右区间较小的一个
 *
 * 使用归并排序
 *
 * 先说第3种怎么求，归并排序中最后一步有一个合并的过程。
 * 两个已经排好序的左区间和右区间进行合并，当我一旦可以找到右区间上的插入临时数组了
 * 也就意味着
 * 1. 当前右区间上的这个数(指针j)比左区间上当前指针i后的所有数都小
 * 2. 当前右区间上的这个数(指针j)比左区间上当前指针前的所有数都大
 * 所以，右区间每出现一个数，则意味着有mid - i + 1个数跟他组成了逆序对
 * 最后我只要统计有x个这样的数，乘(mid - i + 1)就行
 *
 * 那么前两种怎么做？
 * 这样想：归并排序递归到最后，区间就是1个数，那么前两种形式的逆序对根本不存在。
 * 回溯一层，区间呈顺序状后，自然也不存在前两种形式的逆序对。故：
 * 我们只需要在拆分区间的递归回溯中把第三种的数字返回，就是说
 * 第1. 2.种num == 下层递归中 第3.种的数量
 * 所以我们只要维护第三种的数量做递归，最终答案大区间的逆序对个数
 */

#include <iostream>

using namespace std;

typedef long long LL;
const int N = 1e6 + 10;

int n;
int q[N], temp[N];

LL merge_sort_reverse(int q[], int l, int r)
{
    if (l >= r) return 0;

    int mid = (l + r) >> 1;
    LL res = merge_sort_reverse(q, l, mid) + merge_sort_reverse(q, mid + 1, r);

    int i = l, j = mid + 1, k = 0;
    while (i <= mid && j <= r)
    {
        if (q[i] <= q[j]) temp[k++] = q[i++];
        else
        {
            temp[k++] = q[j++];
            res += mid - i + 1;
        }
    }
    while (i <= mid) temp[k++] = q[i++];
    while (j <= r) temp[k++] = q[j++];
    for (int i = 0, j = l; j <= r; i++, j++) q[j] = temp[i];

    return res;
}

int main()
{
    scanf("%d", &n);
    for (int i = 0; i < n; i++) scanf("%d", &q[i]);

    cout << merge_sort_reverse(q, 0, n - 1) << endl;

    return 0;
}
```

