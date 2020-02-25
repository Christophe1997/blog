---
title: "Some Bit Tricks"
date: 2020-02-25T11:50:04+08:00
draft: false
categories: ["Notes"]
tags: ["Bitwise"]
---

## Bitwise tricks

近来翻到一本"Hackers Delight"的书, 其主要介绍基于二进制运算的算法. 初读来大感震撼, 其结果之巧妙,过程之精简, 于仅仅是了解计算机数是用补码所表示, 并未深入了解过二进制运算的人而言不可谓不精美. 正如Dijkstra所言, 计算机程序是集逻辑美感与机械实现的矛盾体. 本文姑且将其中的一些皮毛摘录如下, 以便日后之使用, 目前倒是难以得到应用.

1. 使最右位的1变成0

   ```c#
   // 01011000 -> 01010000
   // if no one then return 0
   x & (x - 1)
   ```

   上式也可以用来判断一个无符号的整数是否是$ 2^n $的形式.

2. 使最右位的0变成1

   ```c#
   // 10100111 -> 10101111
   // if no zero then return -1
   x | (x + 1)
   ```

3. 使尾部的1变成0

   ```c#
   // 10100111 -> 10100000
   // if no trailing 1s then identity
   x & (x + 1)
   ```

   上式也可以用来判断一个无符号的整数是否是$ 2^n - 1 $的形式.

4. 使尾部的0变成1

   ```c#
   // 10101000 -> 10101111
   // if no trailing 1s then identity
   x | (x - 1)
   ```

5. 使最右位的0变成1, 同时其余位变成0

   ```c#
   // 10100111 -> 00001000
   // if no rightmost 0 then return 0
   ~x & (x + 1)
   ```

6. 使最右位的1变成0, 同时其余位变成1

   ```c#
   // 10101000 -> 11110111
   // if no rightmost 1 then return -1
   ~x | (x - 1)
   ```

7. 使尾部的0变成1, 同时其余位变成0

   ```c#
   // 01011000 -> 00000111
   // if no trailling 0s then return 0
   ~x & (x - 1)
   ```

8. 使尾部的1变成0, 同时其余位变成1

   ```c#
   // 10100111 -> 11111000
   // if no trailing 1s then return -1
   ~x | (x + 1)
   ```

9. 只保留最右位的1, 其余位变成0

   ```c#
   // 01011000 -> 00001000
   // if no rightmost 1 then return 0
   x ^ (x - 1)
   ```

10. 使得从右往左到最右位的0都变成1, 其余位变成0

    ```c#
    // 01010111 -> 00001111
    // if no rightmost 0 then return -1
    // else if no trailing 1s then return 1
    x ^ (x + 1)
    ```

11. 使得最右边连续的1变成0

    ```c#
    ((x | (x - 1)) + 1) & x
    ```

    上式也可以用来判断一个无符号的整数是否是$ 2 ^ n - 2 ^ m (n > m) $的形式.

