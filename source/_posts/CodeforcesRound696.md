---
title: '[杂题] Codeforces Round #696 C,D题'
date: 2021-01-23 14:36:20
math: true
tags: [acm, 杂题]
---

##  [C - Array Destruction](https://codeforces.com/contest/1474/problem/C)

**忘记开桶**导致复杂度多了一层n，回头整理一下qwq

### 题目大意

给出 $2n$ 个数，需要造一个数 $x$ 使得满足以下操作：

1. 在数列中任取两数，使其和为 $x$
2. 令 $x$ 变为上面两数中较大者，将这两数扔出数列
3. 重复上述步骤至序列为空

问是否存在这样的数 $x$ 使上述操作可以进行。

数据范围： $n \leq 1000$ ，给出数字 $a_i \leq 1e6$  。

### 解题思路

肯定从大的下手诶（**所以要先排一下序**

不然，如果 $x$ 小于数列中任何一个数，都会导致这个数无法被消除。

第一次选的两个数肯定包含最大的那个数，**枚举一下另一个数**|･ω･｀)

**开个桶**记录一下每个数字的出现次数，模拟上述过程看看能不能行就好啦

好像挺简单..感觉要把代码写得优雅还挺难的。。

### 代码

输入格式：T组。每组：n，之后2n个数。

输出格式：YES或NO，YES的话需要接着输出一开始的数 x 和每次选的两个数。

```cpp
#include <cstdio>
#include <iostream>
#include <algorithm>
#include <stack>
#define rep(i, s, t) for(int i = s; i <= t; i++)
#define per(i, s, t) for(int i = s; i >= t; i--)
#define mp(x, y) make_pair(x, y)
using namespace std;
const int MAXN = 1e3 + 233;
stack <pair<int, int> > stk; // stk用于记录操作路径
int T, n, a[MAXN << 1], tms[1000233]; //tms是桶
bool dfs(int dep, int* now)
{
    if (dep == n) //操作n次即为成功
        return 1;
    int sum = *now; //找出两个数使其和为sum
    while (!tms[*now]) //其中一个数一定是当前序列中存在的最大的，倒着找
        now--;
    if (*now << 1 == sum && tms[*now] < 2) //特判，选的两个数相等的时候够不够选
    	return 0;
    if (tms[sum - *now]) //看看另一个数存不存在
    {
        tms[*now]--;
        tms[sum - *now]--;
        bool flag = dfs(dep + 1, now);
        tms[*now]++;
        tms[sum - *now]++;
        if (flag)
        {
            stk.push(mp(sum - *now, *now));
            return 1;
        }
        else
            return 0;
    }
    else
        return 0;
}
int main()
{
    scanf("%d", &T);
    while (T--)
    {
        scanf("%d", &n);
        rep (i, 1, n << 1)
        {
            scanf("%d", &a[i]);
            tms[a[i]]++;
        }
        sort(a + 1, a + 2 * n + 1);
        bool flag = 0;
        tms[a[n * 2]]--;
        per (i, n * 2 - 1, 1) //从大到小枚举第一次第二个要选的数
        {
            tms[a[i]]--;
            if (dfs(1, &a[n * 2]))
            {
                stk.push(mp(a[i], a[n * 2]));
                flag = 1;
                break;
            }
            tms[a[i]]++;
        }
        if (flag)
        {
            printf("YES\n");
            printf("%d\n", stk.top().first + stk.top().second);
            while (!stk.empty())
            {
                printf("%d %d\n", stk.top().first, stk.top().second);
                stk.pop();
            }
        }
        else
            printf("NO\n");
        rep (i, 1, n << 1) //这里为下一组数据初始化一下tms，为了比memset快点qwq
        	tms[a[i]] = 0;
    }
    return 0;
}
```

感觉这题需要考虑各种各种优化，少写一点就很不优雅QAQ

咱要告诉自己记得**开桶**。。

multiset好像很厉害的样子，学C++到STL的时候补一下叭。~~才不是现在懒得补~~

## [D - Cleaning](https://codeforces.com/contest/1474/problem/D)

### 题目大意

给定一个序列长n，任意次选择相邻的两个数同时减一。开始之前可以任取两个相邻的数互换位置。

问是否可以全变成零。

数据范围: $n \leq 2e5$  数字 $a_i \leq 1e9$

### 解题思路

怎么有种似曾相识的感觉..

就算说是瞎选相邻的，反正最后都要变成0，跟按顺序来是一样的。

正着来的话，第二堆减第一堆，第三堆减第二堆剩下的... $pre[i]$ 表示进行完 i 位置的操作后，i 位置剩余的数字。如果出负数的话之后就不能操作了，拿INF标记一下。

倒着也来一遍，**注意这里INF不能相同**。

然后看看哪个位置能作为两头的汇合点，详见代码啦qwq

### 代码

输入格式：T 组，每组 n，之后 n 个数。

输出格式：YES或NO

```cpp
#include <cstdio>
#include <iostream>
#define rep(i, s, t) for(int i = s; i <= t; i++)
#define per(i, s, t) for(int i = s; i >= t; i--)
#define INF 0x3f3f3f3f
using namespace std;
const int MAXN = 2e5 + 233;
int n, T;
int a[MAXN], pre[MAXN], suf[MAXN];
int main()
{
    scanf("%d", &T);
    while (T--)
    {
        scanf("%d", &n);
        rep (i, 1, n)
            scanf("%d", &a[i]);
        pre[0] = 0, suf[n + 1] = 0; //初始化俩数就行啦
        rep (i, 1, n)
            if (pre[i - 1] > a[i])
                pre[i] = INF;
            else
                pre[i] = a[i] - pre[i - 1];
        per (i, n, 1)
            if (suf[i + 1] > a[i])
                suf[i] = INF << 1; //换标记
            else
                suf[i] = a[i] - suf[i + 1];
        bool flag = 0;
        rep (i, 1, n - 1)
            if (pre[i] == suf[i + 1] || a[i] >= suf[i + 2] && a[i + 1] >= pre[i - 1] && a[i] - suf[i + 2] == a[i + 1] - pre[i - 1])  //直接能操作或者交换之后可以操作，记得这里也要判负数
            {
                flag = 1;
                break;
            }
        if (flag)
            printf("YES\n");
        else
            printf("NO\n");
    }
    return 0;
}
```

