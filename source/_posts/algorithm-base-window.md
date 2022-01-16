---
title: 算法-滑动窗口
date: 2020-01-25 15:12:03
tags:
- algorithm
- cs
categories:
- 其他
---

# 简介

双指针用法的一种，这个算法技巧的思路非常简单，就是维护一个窗口，不断滑动，然后更新答案。

代码结构：

```java
int left = 0, right = 0;

while (right < s.size()) {`
    // 增大窗口
    window.add(s[right]);
    right++;

    while (window needs shrink) {
        // 缩小窗口
        window.remove(s[left]);
        left++;
    }
}
```

# 示例

给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。

```java
public String minWindow(String s, String t) {
    if (s == null || t == null || s.length() < t.length()) {
        return "";
    }

    int left = 0, right = 0, minLeft = 0, minRight = Integer.MAX_VALUE;
    // 目标范围
    Map<Character, Integer> need = new HashMap<>();
    for (int i = 0; i < t.length(); i ++) {
        char c = t.charAt(i);
        need.put(c, need.get(c) == null ? 1 : need.get(c) + 1);
    }
    // 滑动窗口
    Map<Character, Integer> window = new HashMap<>();
    // valid
    int validCount = 0;

    while (right < s.length()) {
        // 扩大窗口
        char c = s.charAt(right);
        right ++;
        if (need.containsKey(c)) {
            window.put(c, window.get(c) == null ? 1 : window.get(c) + 1);
            if (window.get(c).equals(need.get(c))) {
                validCount ++;
            }
        }

        // 缩小窗口
        while (validCount == need.size()) {
            // 更新长度
            if (right - left < minRight - minLeft) {
                minLeft = left;
                minRight = right;
            }
            char cc = s.charAt(left);
            left ++;
            if (need.containsKey(cc)) {
                if (Objects.equals(window.get(cc), need.get(cc))) {
                    validCount --;
                }
                window.put(cc, window.get(cc) - 1);
            }
        }
    }

    return minRight == Integer.MAX_VALUE ? "" : s.substring(minLeft, minRight);
}
```