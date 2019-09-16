## 二进制系统

常见的进制系统有二进制、八进制、十进制、十六进制，概念大家都懂而重点在于进制间的转换和运算，实际上转换和计算都是通过计算机完成，所以建议是理解并可以手写转换和运算然后通过计算机验证，其中在`JS`里验证的方法很简单。

`toString`方法用于进制间的转换，但要记住结果返回的是字符串：

```js
// 把数字转换成你想要的进制表示，例如把十进制的100转换为十六进制
(100).toString(16) === "64";
// 反过来，把十六进制的0x64转换为十进制
(0x64).toString(10) === "100";
```

`parseInt`方法用于解析某个字符串或数字到十进制，前提是你已经知道目标的进制是多少，但要记住结果返回的是数字：

```js
// 默认以十进制读取目标：
parseInt(64) === 64;

// 除非你标识目标的进制类型，例如：
parseInt(64, 16) === 100;
parseInt(0x64) === 100;
parseInt("0x64") === 100;
```

### 其他进制转十进制

技巧就是通过基数把进制进行分解，用 JS 语法来分解，其中`**`为幂符号，并且`0`的幂都是`1`;

二进制的`101010`，基数为`2`，可以分解为：

```js
1 * 2 ** 5 === 32;
0 * 2 ** 4 === 0;
1 * 2 ** 3 === 8;
0 * 2 ** 2 === 0;
1 * 2 ** 1 === 2;
0 * 2 ** 0 === 0;

32 + 0 + 8 + 0 + 2 + 0 = 42;
```

八进制的`33`，基数为`8`，可以分解为：

```js
3 * 8 ** 1 === 24;
3 * 8 ** 0 === 3;

24 + 3 === 27;
```

十六进制的`1B`，基数为`16`，可以分解为：

```js
01 * 16 ** 1 === 16;
11 * 16 ** 0 === 11;

16 + 11 === 27;
```

### 十进制转二进制

这个会难一些而且方法有很多，其中最容易的一个方法是重复除以`2`并记录余数，一直到`0`为止，例如十进制的`245`：

| 取整 | 取余 |
| ---- | ---- |
| 122  | 1    |
| 61   | 0    |
| 30   | 1    |
| 15   | 0    |
| 7    | 1    |
| 3    | 1    |
| 1    | 1    |
| 0    | 1    |

再从下往上组合就得二进制的结果`11110101`，理解原理后通过简单的`JS`递归也可以转换：

```js
function decimalToBinary(num, result = []) {
  const round = Math.floor(num / 2);
  result.unshift(num % 2);
  if (round === 0) {
    return result.join("");
  } else {
    return decimalToBinary(round, result);
  }
}

decimalToBinary(245) === "11110101";

// 验算一下
(245).toString(2) === "11110101";
```

### 十六进制和二进制互转

从二进制转换为十六进制和十六进制转换为二进制实际上非常容易，这是因为十六进制是`16`的基数，实际上是`2`的`4`次幂，只要把逐个字符看成十进制(其中`A=10`、`B=11`类推)，再转为`4`个字符的二进制，再组合即可。

例如十六进制的`75C`转为二进制：

```js
7 === 0b0111;
5 === 0b0101;
12 === 0b1100;

0x75c === 0b011101011100;
```

反过来二进制的`011101011100`转为十六进制：

```js
0b0111 === 7;
0b0101 === 5;
0b1100 === 12;

0b011101011100 === 0x75c;
```

### 八进制和二进制互转

这个和上面一样道理，只要把逐个字符看成十进制，再转为`3`个字符的二进制，再组合即可，不再累述。

### 二进制负数

二进制负数的表示会相对复杂，其中它有两种表示方法，先说第一种就是用最左边的一个符号表示正负。例如我们在八位的二进制里，可以约定最左边的一位：`0`表示数字为正数，`1`表示数字为负数。

根据约定，就有一下结果：

```js
1000011 === -3;
0000011 === 3;
```

想一下，假如没有这个正负标记的时候，八位二进制的表示范围为：

```js
// 最小值为0，最大值为255，总共有256个可能的数字
00000000 === 0;
11111111 === 255;
```

有了正负标记之后，八位二进制的表示范围为：

```js
// 最小值为-127，最大值为127，总共有255个可能的数字
11111111 === -127;
10000000 === 0;
00000000 === 0;
01111111 === 127;
```

那么问题就来了，使用这种表示法，那就要指定我们的数值是多少位，上面是`8`位，也有`16`位和`32`位的情况，假如使用的是`32`位的时候，我想表示一个值为`-1`时，就必须这样写：

```js
10000000000000000000000000000001 === -1;
```

这种表示方法很累赘，那么能不能把正负标记放在最右边呢，那就不会造成存在上面这种现象，答案是可以但没必要，那样不单会造成人读时理解的困难，计算机所需的处理也会增加。

第二种就是用补码去表示负数，简单概况就是负数的二进制表示法为，取其正数，再取其反码，再加一即可。

例如用八位二进制表示`-5`时：

```js
// 首先取正数
5;

// 再转换为二进制
101;

// 再填充到所需的位数
00000101;

// 再反转数字
11111010;

// 最后加1，完成
11111011;
```

问题是为什么要这样做，网上有很多解释，这篇说的很好：[补码原理——负数为什么要用补码表示](https://blog.csdn.net/leonliu06/article/details/78685197)

那么问题又来了，这个二进制的`11111011`，我怎么知道你是正数的`251`还是负数的`-5`呢？那就要看计算机中是以有符号进行存储还是无符号进行存储，如果是无符号存储，则其为一个正数`251`，若是有符号存储，则为补码存储也就是负数的`-5`。

假如我知道`11111011`是有符号存储的，那我就反过来求十进制的负数吧：

```js
// 首先减一
11111010;

// 再反转数字
00000101;

// 换十进制
5;

// 取负数，完成
-5;
```

再回头看，假如使用的是`32`位的时候，我想表示一个值为`-1`时，使用补码写法：

```js
// 首先取正数
1;

// 再转换为二进制
1;

// 再填充到所需的位数
000000000000000000000001;

// 再反转数字
111111111111111111111110;

// 最后加1，完成
111111111111111111111111;
```

假如你不知道有没符号的情况下就去取值，可能会表示出一个偏差相当大的一个数，所以要特别注意。

在`JS`里值得注意的一个是`toString`不会按上面预期的返回补码表示法：

```js
(-5).toString(2) === "-101";
```

对比一下上面的转换步骤，会发现`JS`把取反码和加一这两个步骤用一个负号取代替了。