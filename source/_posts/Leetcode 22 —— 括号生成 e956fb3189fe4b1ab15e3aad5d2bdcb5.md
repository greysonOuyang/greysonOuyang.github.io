---
title: Leetcode 22 —— 括号生成
date: 2021-04-17 21:22:36
tags: leetcode,算法
---


思路：

回溯

![Leetcode%2022%20%E2%80%94%E2%80%94%20%E6%8B%AC%E5%8F%B7%E7%94%9F%E6%88%90%20e956fb3189fe4b1ab15e3aad5d2bdcb5/Snipaste_2020-03-06_16-37-03.png](Leetcode%2022%20%E2%80%94%E2%80%94%20%E6%8B%AC%E5%8F%B7%E7%94%9F%E6%88%90%20e956fb3189fe4b1ab15e3aad5d2bdcb5/Snipaste_2020-03-06_16-37-03.png)

实现：

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        // res记录结果
        List<String> res = new LinkedList<String>();
        helper("", res, n, 0, 0);
        return res;
    }
    
    // cur传入当前String状态、left是已有左括号，right是已有右括号
    public void helper(String cur, List<String> res, int n, int left, int right) {
        // 右括号等于给定数n，则返回List
        if (right == n) {
            res.add(cur);
            return;
        }
        // 左括号少于n个，可以添加左括号
        if (left < n) {
            helper(cur + "(", res, n, left + 1, right);
        }
        // 右括号少于左括号，可以添加右括号
        if (right < left) {
            helper(cur + ")", res, n, left, right + 1 );
        }
    }
}
```