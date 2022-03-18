---
title: '[模板]LCS最长公共子序列'
date: 2022-03-18 10:31:59
tags: [acm, 模板, LCS]
---

 [luoguP1439](https://www.luogu.com.cn/problem/P1439)，本身不算难，记一下防止自己下次看到不会做qwq

## 题意

给出 1 到 n 的两组全排列，求它们的最长公共子序列。

## 最长上升子序列LIS

朴素做法：f\[i\] 到第 i 个元素的LIS长度，n方做即可。

二分做法：f[i] 表示长度为 i 的上升子序列，每次对于一个新的元素，二分查出比这个值大的第一个值，去更新 f 数组。

```cpp
rep (i, 1, n)
{
	int l = 0, r = mx, mid;
    // 可以扩展最大长度
	if (a[i] > f[mx])
		f[++mx] = a[i];
	else
	{
		while (l < r)
		{
			mid = (l + r) >> 1;
			if (f[mid] > a[i])
				r = mid;
			else
				l = mid + 1;
		}
        // 更新该长度对应的最小值
		f[l] = min(f[l], a[i]);
	}
}
```

## 最长公共子序列LCS

a，b 两数组对应的元素相同，位置不同。

直接找 b 数组每个元素对应在 a 数组中的位置，求这个位置的LIS即可。

```cpp
#include <cstdio>
#include <iostream>
#define rep(i,s,t) for(int i=s;i<=t;i++)
using namespace std;
const int MAXN = 1e5 + 233;
int n, a[MAXN], b[MAXN], pos[MAXN], f[MAXN];
int main()
{
	// freopen("in.txt", "r", stdin);
	scanf("%d", &n);
	rep (i, 1, n)
	{
		scanf("%d", &a[i]);
		pos[a[i]] = i; // pos记录每个数字在a中的位置
		f[i] = 0x3f3f3f3f;
	}
	rep (i, 1, n)
		scanf("%d", &b[i]);
	int mx = 0;
	rep (i, 1, n)
	{
		int l = 0, r = mx, mid;
		if (pos[b[i]] > f[mx])
			f[++mx] = pos[b[i]];
		else
		{
			while (l < r)
			{
				mid = (l + r) >> 1;
				if (f[mid] > pos[b[i]])
					r = mid;
				else
					l = mid + 1;
			}
			f[l] = min(f[l], pos[b[i]]);
		}
	}
	printf("%d\n", mx);
	return 0;
}
```

