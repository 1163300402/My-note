# LeetCode-刷题之旅

## 动态规划
### 53.最大自序和（easy）
[53.最大自序和](https://leetcode-cn.com/problems/maximum-subarray/description/)

[思路正确，代码有问题](https://blog.csdn.net/zx582727090/article/details/52174182)

我的方法
```java
class Solution {
    public int maxSubArray(int[] nums) {
        int max= Integer.MIN_VALUE;
        int cur= 0;
        
        if (nums.length== 0)
            return 0;
        for (int i=0; i<nums.length; i++){
            cur+= nums[i];
            if (cur> max )
                max= cur;
            if (cur< 0)
                cur= 0;
        }
        return max;
    }
}
```
[三种方法](https://blog.csdn.net/sinat_30440627/article/details/54924737)

最快的方法
```java
class Solution{
    public int maxSubArray(int[] nums) {
        int length= nums.length;
        if (length== 0)
            return 0;
        int max= nums[0];
        int cur= nums[0];
        for (int i=1; i<length; i++){
            if (cur>0){
                cur+= nums[i];
            }else{
                cur= nums[i];
            }

            if(cur> max)
                max= cur;
        }
        return max;
    }
}
        
```