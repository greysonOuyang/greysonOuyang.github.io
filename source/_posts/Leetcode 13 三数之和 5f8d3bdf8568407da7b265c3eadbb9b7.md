---
title: Leetcode 13 三数之和
date: 2021-04-17 21:22:36
tags: leetcode,算法
---

思路：

一个Base+双指针

先排序，base先指向第一个数，left指向base之后的第一个数，right指向最后一个数，sum>0，right向左移，sum<0，left向右移，找出数num+left所指数=target-base；找到了则base右移，注意去重，即指针移动时跳过相等的数

实现：

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        ArrayList<List<Integer>> res = new ArrayList<List<Integer>>();
        if (nums == null || nums.length <= 2) return res;
        int n = nums.length;
        int i = 0;
        Arrays.sort(nums);

        while (i <= n - 2) {
            int base = nums[i];
            int left = i+1;
            int right = n-1;

            while (left < right) {
                int sum = base + nums[left] + nums[right];
                if (sum == 0) {
                    LinkedList<Integer> list = new LinkedList<Integer>();
                    list.add(base);
                    list.add(nums[left]);
                    list.add(nums[right]);
                    res.add(list);
                    left = moveRight(nums, left + 1);
                    right = moveLeft(nums, right - 1);
                } else if (sum > 0) {
                    right = moveLeft(nums, right - 1);
                } else {
                    left = moveRight(nums, left + 1);
                }
            }
            i = moveRight(nums, i + 1);
        }
        return res;
    }
    public int moveLeft(int[] nums, int right) {
        while (right == nums.length - 1 || (right >= 0 && nums[right] == nums[right + 1])) {
            right --;
        }
        return right;
    }
    public int moveRight(int[] nums, int left) {
        while (left == 0 || (left < nums.length && nums[left] == nums[left - 1])) {
            left ++;
        }
        return left;
    }
}
```

时间复杂度：O(N2)