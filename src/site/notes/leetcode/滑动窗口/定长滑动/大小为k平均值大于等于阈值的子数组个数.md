---
{"dg-publish":true,"dg-permalink":"/leetcode/滑动窗口/大小为k平均值大于等于阈值的子数组个数","tags":["滑动窗口"],"permalink":"/leetcode/滑动窗口/大小为k平均值大于等于阈值的子数组个数/","dgPassFrontmatter":true}
---



![Pasted image 20250325115105.png](/img/user/%E9%99%84%E4%BB%B6/Pasted%20image%2020250325115105.png)

```cpp
class Solution {
public:
    int numOfSubarrays(vector<int>& arr, int k, int threshold) {
        int ans = 0;
        int s = 0; // 维护窗口元素和
        for (int i = 0; i < arr.size(); i++) {
            // 1. 进入窗口
            s += arr[i];
            if (i < k - 1) { // 窗口大小不足 k
                continue;
            }
            // 2. 更新答案
            ans += s >= k * threshold;
            // 3. 离开窗口
            s -= arr[i - k + 1];
        }
        return ans;
    }
};
```
