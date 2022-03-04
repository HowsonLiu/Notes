# 旋转排序数组
## 思路
旋转排序数组的一个特点是：经过旋转之后，它是部分有序的。由于不知道旋转的值是多少，因此我们不能确定零点的位置
> 例如：
`[0, 1, 2, 3, 4]`
可以旋转成
`[1, 2, 3, 4, 0]`
也可以旋转成
`[4, 0, 1, 2, 3]`

但是我们可以发现一个规律：**将数组从中间一份为二之后，其中一边肯定是有序的，而另一边则是另一个旋转排序数组。**

根据这个特性，我们可以**对有序的半边进行普通的二分查找**，而**对另一边进行递归处理（不处理）**

```c++
int left = 0, right = n-1;
while(left < right) {
    int mid = left + (right-left)/2;
    if(nums[mid] < nums[right]) {
        // 右边是有序的
        ...
    }
    else if(nums[mid] > nums[right]) {
        // 左边是有序的
        ...
    }
    else if(nums[mid] == nums[right]) {
        // 不能确定右边是否有序
    }
}
```

出于个人习惯，我一般选择看右边是否有序，当然可以选择左边

##  寻找旋转排序数组中的最小值（[I](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)/[II](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/)）
因为我们要找的是零点，因此
- 当右边有序时，零点可能在左边或者恰好在中间，因此我们需要缩短右边界`right = mid`
- 当左边有序时，零点不可能在左边或者中间，因此我们需要缩短左边界`left = mid+1`
- 当不能确定右边是否有序时，因为`nums[mid] == nums[right]`，简单缩短右边边界一格并不会引起结果的改变，因此我们缩短右边界`right--`

## 搜索旋转排序数组（[I](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)/[II](https://leetcode-cn.com/problems/search-in-rotated-sorted-array-ii/)）
我们要找的值是`target`，因此
- 若左中右三值其中一值等于`target`，可以直接返回下标
- 当右边有序时
    - 若`target`在`nums[mid]`与`nums[right]`间，则我们缩短左边界`left = mid+1`
    - 否则缩短右边界`right = mid-1`
- 左边有序
    同理
- 不能确定右边是否有序，简单缩短右边界一格

需要注意的是，右边有序时，**并不能简单通过比较`nums[mid]`来确定`target`在右边界**。因为这是一个循环数组，**有可能`target`比`nums[mid]`和`nums[right]`都大**，那么`target`就在左边了。因此`target > nums[mid] && target < nums[right]`不能省
