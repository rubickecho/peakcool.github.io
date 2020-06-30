---
author: tangliangcheng
comments: true
date: 2020-06-30 21:51:00+00:00
layout: post
slug: JavaScript Array.from() VS Array.fill()
title: JavaScript Array.from() VS Array.fill()
excerpt: JS Array.fill() 踩坑
categories:
- 技术分享
---

## JavaScript Array.from() 和 Array.fill() 踩坑

今天在解决一道算法题时，需要声明一个二维数组，于是，便有了下面的解法

```js
//类背包问题，题目转换为：恰好存在一种装法，一个包正好装（总价值 / 2）的物品
//状态：容量、价值
//条件：dp[i][j] = sum / 2 => true
//状态转移方程: dp[i][j] = dp[i-1][j] + dp[i-1][j - nums[i-1]]
function canPartition(nums: number[]): boolean {
    let sum:number = 0;
    nums.forEach((v) => { sum += v }); //计算总值
    
    if (sum % 2 !== 0) return false; //奇数和时
    const n:number = nums.length;
    const halfSum:number = sum / 2;

    let dp1: Array<Array<boolean>> = Array(n + 1).fill(Array(halfSum + 1).fill(false));
    // let dp2: Array<Array<boolean>> = Array.from({ length: n + 1 }, () =>
    //     new Array(halfSum + 1).fill(false));

    console.log('dp2', JSON.stringify(dp2))
    // console.log('dp1', JSON.stringify(dp1))

    /**----------- */
    var dp = dp1 // => 最终return true
    // var dp = dp2 // => 最终return false
    /**----------- */

    for(let i: number = 0; i <= n; i++) {
        dp[i][0] = true;
    }

    for(let i: number = 1; i <= n; i++) {
        for(let j:number = 1; j <= halfSum; j++) {
            if (j - nums[i - 1] >= 0) {
                dp[i][j] = dp[i - 1][j] || dp[i - 1][j - nums[i - 1]];
            } else {
                dp[i][j] = dp[i - 1][j];
            }
        }
    }
    return dp[n][halfSum];
};

console.log(canPartition([2, 2, 3, 5]));
```

执行该解法，传入`[2, 2, 3, 5]`时，期望应该是`return false`，事与愿违，按照我上面的逻辑是`return true`，于是，便开始了踩坑排雷。

## 声明二维数组

在JS中要先声明二维数组，我们先来看`如何声明一维数组`。

```js
const array1 = new Array()

const array2 = new Array(5) //声明长度为5的数组

const array3 = new Array(5).fill(1) //声明长度为5，并用1填充满的数组

const array4 = new Array().from({length}, function() {return 1}) //同样声明长度为5，并用1填充满的数组

```

那么二维就简单了
```js
const array5 = new Array(5).fill(new Array(4).fill(1))

const array6 = new Array().from({length: 5}, function() {
    return new Array(4).fill(1);
})
```

所以可以看到，在上方解法中，我注释掉的 `dp2`，就是 `array6`式的声明方式，而用这种方式声明的二维数组，执行`canPartition([2, 2, 3, 5])`，就可以得到正确的结果，为什么呢？

**因为 问题出在 Array.fill()**

## Array.from() 和 Array.fill()


我做了如下代码测试两种声明方式


```js
const a1 = Array(2).fill({name: ''});
console.log('a1', a1) // [{name: ''}, {name: ''}]
a1[0].name = 'Jack'
console.log('a1', a1) // [{name: 'Jack'}, {name: 'Jack'}]

const b1 = Array(2).fill([1]);
console.log('b1', b1) // [1, 1]
b1[0] = 2
console.log('b1', b1) // [2, 1]

const b2 = Array.from({length: 2}, function(){return [1]});
console.log('b2', b2) // [[1], [1]]
b2[0][0] = 2
console.log('b2', b2) // [[2], [1]]

```

以下是MDN对两个方法的描述
```
Array.from() 方法从一个类似数组或可迭代对象创建一个新的，浅拷贝的数组实例。

Array.fill() 方法用一个固定值填充一个数组中从起始索引到终止索引内的全部元素。不包括终止索引。
```

所以，由`Array.fill()`填充的值如果是`object`类型，则所有值都会指向一个内存地址，不管更新那一个，都将影响其他值

而`Array.from()`是浅拷贝，即每个`object`值都指向不同的引用地址，更新值都只会影响该值自己


## 最后

所以，平时写代码时，如果只填充`基本类型`的值，则可以使用`Array.fill()`；如果需要填充`引用类型`的值，则只能使用`Array.from()`

### 参考网址

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/fill

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from

