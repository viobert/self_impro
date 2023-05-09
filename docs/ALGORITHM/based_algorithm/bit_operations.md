# 位运算

## 简介

## 常见操作

介绍几种位运算常见的操作
1. 一个数`n`的二进制下第`k`位是`0`还是`1 `            `n >> k & 1`

2. `LowBit(n)`操作  返回`n`的二进制下最后（最右边的）一位`1`       `n & (~n + 1)`

    ​	解释：

    + n == ..............`1(最后一位1)00000`
    +  ~n == ..............`011111`
    +   ~n + 1 == ..............`100000`

## 常见操作的Coding

```c++
#include <iostream>

using namespace std;

const int N = 1e5 + 10;
int n;
int q[N];

int num_binary1(int x)
{
    int num = 0;
    while (x)
    {
        num += x & 1;
        x = x >> 1;
    }
    return num;
}

int main()
{
    scanf("%d", &n);
    for (int i = 0; i < n; i++) scanf("%d", &q[i]);

    for (int i = 0; i < n; i++) printf("%d ", num_binary1(q[i]));
    printf("\n");
    return 0;
}
```

## Correlation

### 1 快速幂

#### 简介

快速幂，二进制取幂（Binary Exponentiation，也称平方法），是一个在$O(\log(n))$的时间内计算$a^n$的小技巧，而暴力的计算需要$O(n)$的时间。

#### Method

快速幂的思想就是，将**指数**变成二进制，做二进制运算。

1. 值（0或者1）代表做乘法与否
2. 位数表示$2^n$，每次逐渐平方。

$1101011$    $0/1$正好可以表示乘不乘，位数正好表示偶数次方：从后往前就是一次平方，最后一位是$a$，倒数第二位是$a^2$，倒数第三位是$a^4$

**也就是说，每次都平方一下，然后0/1判断乘不乘上去即可。**

### Code

```c++
#include <iostream>

using namespace std;

#define LL long long

LL quick_exponentiation(LL a, LL b, LL q)
{
    LL res = 1, temp = a;
    while(b)
    {
        if (b & 1) res = res * temp % q;
        temp = temp * temp % q;
        b >>= 1;
    }
    return res % q;  // %一下卡掉样例 1 0 1
}

int main ()
{
    int a, b, q;
    cin >> a >> b >> q;

    printf("%lld\n", quick_exponentiation(a, b, q));
    return 0;
}
```

