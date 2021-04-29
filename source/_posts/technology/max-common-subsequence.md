---
title: 提取两个字符串的最大公共子串
author: chiechie
mathjax: true
date: 2021-03-16 23:46:13
tags:
- 编程
- 数据结构
categories: 
- 技术
---

## 题目和思路
题目：求两个字符串的最大公共子串。
这个公共子串可以是连续的也可以是不连续的，区别不大，下面只看连续的情况。

思路是，构造一个矩阵，行为字符串A的每个字符，列为字符串B的每个字符
矩阵中元素默认为0，
第i行第j列的元素表示，A的子串（index从0到i）和B的子串(index葱从0到j)的最大公共子串的长度，并且这个公共子串在A的index为 ✨ 到i，在B的index为✨到j。

例如
```
输入：text1 = "abcde", text2 = "ace" 
输出：3  
解释：最长公共子序列是 "ace" ，它的长度为 3 。
```
酱紫

### 最简单的实现


```python
import numpy as np
## 不连续子串的版本
class Solution(object):
    def longestCommonSubsequence(self, text1, text2):
        """
        :type text1: str
        :type text2: str
        :rtype: int
        """
        compare_matrix = np.zeros((len(text1) + 1, len(text2) + 1))
        for i, s1 in enumerate(text1):
            for j, s2 in enumerate(text2):
                if s1 == s2:
                    compare_matrix[i + 1, j + 1] = compare_matrix[:(i+1), :(j+1)].max() + 1
        return int(compare_matrix.max())
## 连续子串的版本
class Solution(object):
    def longestCommonSubsequence(self, text1, text2):
        """
        :type text1: str
        :type text2: str
        :rtype: int
        """
        compare_matrix = np.zeros((len(text1) + 1, len(text2) + 1))
        for i, s1 in enumerate(text1):
            for j, s2 in enumerate(text2):
                if s1 == s2:
                    compare_matrix[i + 1, j + 1] = compare_matrix[i, j] + 1
        return int(compare_matrix.max())
```
虽然结果对了，但是耗时太久，消耗内存大。


### 优化了一下运行时间
```python
import numpy as np
class Solution(object):
    def longestCommonSubsequence(self, text1, text2):
        """
        :type text1: str
        :type text2: str
        :rtype: int
        """
        l1 , l2 = len(text1)  , len(text2)
        compare_matrix = [[0 for _ in range(l2 + 1)] for _ in range(l1 + 1)]
        for i in range(l1):
            for j in range(l2):
                if text1[i] == text2[j]:
                    compare_matrix[i + 1][j + 1] = compare_matrix[i][j] + 1
                else:
                    connitues = 0
                    compare_matrix[i + 1][j + 1] = max(compare_matrix[i+1][j], compare_matrix[i][j+1])
        return compare_matrix[i+1][j+1]
```

![img.png](dl-framework/img.png)


## 参考
1. [求两个字符串的连续最大公共子串-leetcode地址](https://leetcode-cn.com/problems/longest-common-subsequence/)
2. [求两个字符串的非连续最大公共子串-youtube](https://www.youtube.com/watch?v=LhpGz5--isw)
3. [smith-waterman-wiki](https://en.wikipedia.org/wiki/Smith%E2%80%93Waterman_algorithm)
4. [alignment-smith-github](https://github.com/alevchuk/pairwise-alignment-in-python/blob/master/alignment.py)