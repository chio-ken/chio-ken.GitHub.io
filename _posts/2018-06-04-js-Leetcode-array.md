

用JavaScript把LeetCode上的数组题做一遍，在这里记录一下。

### 删除有序数组中的重复项

题目描述：

> 给定一个排序数组，你需要在**原地**删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。
>
> 不要使用额外的数组空间，你必须在**原地修改输入数组**并在使用 O(1) 额外空间的条件下完成。



思路：

假如有个数组``` nums = [0, 0, 0, 1, 1, 2]```应该返回新长度3。我的思路就是用两个索引遍历循环数组，发现重复，就把重复的值覆盖掉，只留下所有重复值中的第一个。如下：

```javascript
/**
* @param {number[]} nums
* @return {number}
*/
let removeDuplicates = function(nums) {
    if (nums === null || nums.length === 0) {
        return 0;
    }
    
    let i = 0,
        j = 0;
    for (; i < nums.length; i++) {
        if (nums[j] != nums[i]) {
            nums[++j] = nums[i];
        }
    }
    return j + 1;
};
```

以上面的数组``` nums = [0, 0, 0, 1, 1, 2]```为例：

```javascript
j = 0, i = 0; 这时nums[j] === nums[i]，j保持不变，i加一

j = 0, i = 1; nums[j] === nums[i]，j保持不变，i加一

j = 0, i = 2; nums[j] === nums[i]，j保持不变，i加一

j = 0, i = 3; nums[j] ！= nums[i]了，把nums[i]的值赋给num[j]的下一项。这时的数组就变成了[0, 1, 1, 2]，也就是前三个0最后只剩下了一个。接下来继续重复之前的步骤,最后新数组的项目数就是j + 1了。
```



### 两数之和

题目描述：

> 给定一个整数数组和一个目标值，找出数组中和为目标值的**两个**数。
>
> 你可以假设每个输入只对应一种答案，且同样的元素不能被重复利用。



思路：

两层遍历。从第一层循环的下一个元素开始遍历，符合条件时，返回下标。



```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
    for (var i = 0; i < nums.length; i++) {
        // 第二次可以从第一次遍历所在的位置的下一个元素开始遍历,所以j = i + 1;
        for (var j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] == target) {
                return [i, j];
            }
        }
    }
    return null;
};
```



### 数组拆分

问题描述：

> 给定长度为 **2n** 的数组, 你的任务是将这些数分成 **n** 对, 例如 (a1, b1), (a2, b2), ..., (an, bn) ，使得从1 到 n 的 min(ai, bi) 总和最大。 
>
> **示例 1:** 
>
> ```
> 输入: [1,4,3,2]
> 
> 输出: 4
> 解释: n 等于 2, 最大总和为 4 = min(1, 2) + min(3, 4).
> ```



思路：

最开始的想法就是对原数组进行排序，因为要使输出的总和最大，就要尽量让每一对(a1, b1)中的两个数接近，这样就不会让比较大的元素被”浪费“。

用sort方法进行排序：

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
let arrayPairSum = function(nums) {
    nums.sort();
    let sum = 0;
    for (let i = 0; i < nums.length; i = i + 2) {
        sum += nums[i]; 
    }
    return sum;
};
// test
arrayPairSum([1, 4, 3, 2]); // 4
arrayPairSum([-1, 4, 3, 2]); // 2 
arrayPairSum([10，4, 3, 2]); // 13 [10, 2, 3, 4]
arrayPairSum([-1, -4, 3, 2]); // 1 
                              // nums = [-1, -4, 2, 3]
```

因为sort方法是默认排序是字符串Unicode码点，所以很不稳定。

看来只能手动地进行排序了。= _ =

大体思路就是对数组进行双层遍历，在外层遍历下标等于i时，对i之前的元素进行遍历，如果```nums[j] ```> ```nums[j+1]```，就交换。代买如下：

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
// 交换函数
let swap = function(i, j, nums) {
    let temp = nums[j];
    nums[j] = nums[i];
    nums[i] = temp;
}

let arrayPairSum = function(nums) {
    let len = nums.length;
    for (let i = 0; i < len; i++) {
        isSwap = false;
        for (let j = 0; j < len - i -1; j++) {
            if (nums[j] > nums[j+1]) {
                isSwap = true;
                swap(j, j+1, nums);
            }
        }
        if (!isSwap) {
            break;
        }
    }
    let arr = [];
    for (let i = 0; i< len / 2; i++) {
        arr.push(nums[2 * i]);
    }
    let sum = 0;
    for (let i = 0; i < arr.length; i++) {
        sum += arr[i];
    }
    return sum;
};
```



### 移除元素

问题描述：

> 定一个数组 *nums* 和一个值 *val*，你需要**原地**移除所有数值等于 *val* 的元素，返回移除后数组的新长度。
>
> 不要使用额外的数组空间，你必须在**原地修改输入数组**并在使用 O(1) 额外空间的条件下完成。
>
> 元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。
>
> **示例 1:** 
>
> ```
> 给定 nums = [3,2,2,3], val = 3,
> 
> 函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。
> 
> 你不需要考虑数组中超出新长度后面的元素。
> ```



思路：

这个和删除有序数组重复项的那一题比较相似，我的思路是对数组进行遍历，如果```nums[i]```=```val```，就让```nums[i]```=```nums[i+1]```。

```javascript
/**
 * @param {number[]} nums
 * @param {number} val
 * @return {number}
 */
var removeElement = function(nums, val) {
    var n = nums.length;
    if (nums === null || n === 0) {
        return 0;
    }
    var i = 0;
    for (; i < n; i++) {
        if (nums[i] === val) {
            nums[i] = nums[++i];
            }
    }
    return nums; // [3, 2, 2, 3] => [2, 2, 2, undefined]
};
```

但是出现了问题，因为移除了元素之后，数组的长度是要变化的。

那就尝试在覆盖元素的时候改变数组的长度，每覆盖一个就长度减一。但是如果最后一个元素等于```val```的话就又出现越界问题了。。。

那么试一下先把长度减一，然后将最后一个元素赋值给目标元素。

```javascript
/**
 * @param {number[]} nums
 * @param {number} val
 * @return {number}
 */
var removeElement = function(nums, val) {
    var n = nums.length;
    if (nums === null || n === 0) {
        return 0;
    }
    var i = 0;
    for (; i < n; i++) {
        if (nums[i] === val) {
            n--;
            nums[i] = nums[n];
            }
    }
    return nums; 
};
removeElement([3, 2, 2, 3], 3); // [3, 2, 2, 3]
```

还是出现了问题，分析一下：当```i = 0```时，```nums[0]``` = ```3```，将数组元素减一，末位元素移到第一位，下一步i就变成2了，所以原来的末位元素并没有进行判断。

所以只有判断当前元素不等于3时，才移动下标，往下进行遍历——使用```while```循环。

```javascript
/**
 * @param {number[]} nums
 * @param {number} val
 * @return {number}
 */
var removeElement = function(nums, val) {
    var n = nums.length;
    if (nums === null || n === 0) {
        return 0;
    }
    var i = 0;
    while(i < n) {
        if (nums[i] === val) {
            n--;
            nums[i] = nums[n];
        } else {
            i++;
        }
    }
    return n; 
};
removeElement([3, 2, 2, 3], 3); // [2, 2]
```



### 搜索插入位置

题目描述：

```
给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

你可以假设数组中无重复元素。

示例 1:
输入: [1,3,5,6], 5
输出: 2

示例 2:
输入: [1,3,5,6], 2
输出: 1
```

思路：

首先这个数组是有序的，而且无重复元素，就直接进行遍历，如果当前元素等于```target```，就返回当前下下标，如果比```target```大，也返回当前下标。如果遍历都结束了还是比```target```小，就返回数组长度。

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number}
 */
var searchInsert = function(nums, target) {
    var i = 0,
        len = nums.length;
    if (nums === null || len === 0) {
        return 0;
    }
    
    for (; i < len; i++) {
        if (nums[i] >= target) {
            return i;
        }
    }
    return len;
};
```

为了优化算法，也可以使用```二分法```，这也是面试官想看到的算法。

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number}
 */
let searchInsert = function(nums, target) {
    let end = nums.length - 1,
        start = 0;
    
    while(start <= end) {
        let mid = Math.floor(start + (end - start) / 2);
        if (nums[mid] === target) {
            return mid;
        }
        if (nums[mid] > target) {
            end = mid - 1;
        } else {
            start = mid + 1;
        }
    }
    return start;  
};
```

上面用了```Math.floor()```的原因是，保证赋给```mid```的值是整数，比如```start = 0```，```end = 3```的时候，```mid```就是```1.5```。肯定得不到正确答案。我也是看了半天才注意到，因为算法肯定没有问题，用```Java```验证了一遍。

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int start = 0, end = nums.length - 1;
     
        while(start <= end){
            int mid = start + (end - start) / 2;
            if(nums[mid] == target){
                return mid;
            }
            if(nums[mid] > target){
                 end = mid - 1;
            } else {
                start = mid + 1;
            }
        }
        return start;
    }
}
```

​	

### 存在重复元素

问题描述：

> 给定一个整数数组，判断是否存在重复元素。
>
> 如果任何值在数组中出现至少两次，函数返回 true。如果数组中每个元素都不相同，则返回 false。



思路：

