---
title: Datalab_Share  
date: 2018-04-27 23:19:47  
tags: [CSAPP,binary]
copyright: true
---
# 原料：
* [Writeup](http://csapp.cs.cmu.edu/3e/datalab.pdf)
* [Handout.tar](http://csapp.cs.cmu.edu/3e/datalab-handout.tar)  

# 仓库:
* [博主代码](https://github.com/Darren2017/Lab/tree/master/datalab-handout)

# 题目解析：

## bitAnd
### 要求： 
```
bitAnd - x&y using only ~ and |   
Example: bitAnd(6, 5) = 4 
Legal ops: ~ |   
Max ops: 8 
Rating: 1	 
```
### 函数：
```C
int bitAnd(int x, int y) {
    return ~((~x) | (~y));
}
```
### 解析：
   * 对x y分别取反运算后进行按位或运算，此时若原x y相对应的位有一个不为1， 则按位或之后得到1， 若全为1则得到0，最后再进行取反运算即可达到与按位与相同的结果。

## getByte
### 要求：
```
getByte - Extract byte n from word x
Bytes numbered from 0 (LSB) to 3 (MSB)
Examples: getByte(0x12345678,1) = 0x56
Legal ops: ! ~ & ^ | + << >>
Max ops: 6
Rating: 2
 ```
### 函数：
```C
int getByte(int x, int n) {
    int move = n << 3;
    int result = x >> move;
    result = result & 0xff;

    return result;
}   
```
### 解析:
``n << 3``  即 n * 2 ^ 3 的意思，将x进行右移运算，后用0xff将后八位（所求字节）保留，其余清零。

## logicalShift
### 要求
```
logicalShift - shift x to the right by n, using a logical shift
Can assume that 0 <= n <= 31
Examples: logicalShift(0x87654321,4) = 0x08765432
Legal ops: ! ~ & ^ | + << >>
Max ops: 20
Rating: 3 
```
### 函数
```C
int logicalShift(int x, int n) {
  x = x >> n;
  int handle = (1 << n) + (~1 + 1);
  handle = handle << (32 + (~n + 1));
  handle = ~handle;
  return x & handle;
}
```
### 解析：
首先将x进行算数右移得到初步的x，第一个handle确定有几位需要被覆盖，并用1来表示。将handle左移到需要覆盖的位置后进行取反运算得到掩码，最终进行逻辑与得到结果。由于不能用减法，因此用取反加一的方式得到相反数，用加法实现减法，避开操作限制。

## bitCount
### 要求：
```
bitCount - returns count of number of 1's in word
Examples: bitCount(5) = 2, bitCount(7) = 3
Legal ops: ! ~ & ^ | + << >>
Max ops: 40
Rating: 4
```
### 函数：
```C
int bitCount(int x) {
  x = (x & 0x55555555) + ((x >> 1) & 0x55555555);  
  x = (x & 0x33333333) + ((x >> 2) & 0x33333333);  
  x = (x & 0x0f0f0f0f) + ((x >> 4) & 0x0f0f0f0f);  
  x = (x & 0x00ff00ff) + ((x >> 8) & 0x00ff00ff);  
  x = (x & 0x0000ffff) + ((x >> 16) & 0x0000ffff);  
  return x;
}
```
### 解析：
* emmmm....这道题确实有难度。一开始博主按照最菜的做法，将x右移和1做与运算后计数1的个数，该做法首先效率太低，其次操作次数过多超出要求。
* 修改之后的代码有点复杂可能不是很好理解，此处先附上博主学习本方法的** [链接](https://blog.csdn.net/hitwhylz/article/details/10122617) **。  
* 函数所用思想可以归类为分段的思想。先看一下0x55555555等数字的具体意义：
```
0x5555……这个换成二进制之后就是0101010101010101……
0x3333……这个换成二进制之后就是0011001100110011……
0x0f0f……这个换成二进制之后就是0000111100001111……
```

* 所以可以认为``0x55...``将x划分为两个数字一个周期，有了周期的认识我们可以进一步理解我们是如何计数的，用``x = 011``为例,可以看出我们用第一位与第二位相加即可得到1的数量，那么如何相加呢？给出如下解决方案``cnt = (x >> 1) + (x & 1)``.以此为基础我们看上述代码，``0x55555555``作为一个掩码，覆盖掉每个周期的第一个数字，在每一个周期中达到``（x & 1）``的目的，``((x >> 1) & 0x55555555)``达到``(x >> 1)``的目的，二者相加得到每一个周期中1的数量。
* 计算到现在，我们的每一个周期所代表的值相加可得最终的值，因此我们现在需要做的是将周期进一步扩大，直到32个数字为一个周期，那么后16位的值便是最终结果。而且我们已经利用掩码将前16为屏蔽，故结果容易得出。   

### PS：
这种做法在macOS上无法通过，可以在Ubuntu上跑。

## bang
### 要求：
```
bang - Compute !x without using !
Examples: bang(3) = 0, bang(0) = 1
Legal ops: ~ & ^ | + << >>
Max ops: 12
Rating: 4 
```
### 函数：
```C
int bang(int x) { 
  x = ~ x;
  x = (x & 0x55555555) + ((x >> 1) & 0x55555555);  
  x = (x & 0x33333333) + ((x >> 2) & 0x33333333);  
  x = (x & 0x0f0f0f0f) + ((x >> 4) & 0x0f0f0f0f);  
  x = (x & 0x00ff00ff) + ((x >> 8) & 0x00ff00ff);  
  x = (x & 0x0000ffff) + ((x >> 16) & 0x0000ffff);  
  return x >> 5;
}
```
### 解析：
分析的0与非0的区别为0的非全为1，共有32个，结合bigCount一题，将x取反后计算二进制数中1的个数后除以32即可的结果。

## tmin
### 要求：
```
tmin - return minimum two's complement integer 
Legal ops: ! ~ & ^ | + << >>
Max ops: 4
Rating: 1
```
### 函数：
```C
int tmin(void) {
  return 1 << 31;
}
```
### 解析：
不愧是难度为1的题，求补码二进制最小的数字，将1左移31位即可得到，此处对补码要有一定了解。

## fitsBits
### 要求：
```
fitsBits - return 1 if x can be represented as an 
n-bit, two's complement integer.
1 <= n <= 32
Examples: fitsBits(5,3) = 0, fitsBits(-4,3) = 1
Legal ops: ! ~ & ^ | + << >>
Max ops: 15
Rating: 2
```
### 函数：
```C
int fitsBits(int x, int n) {
  int move = 32 + (~n + 1);
  int result = (x << move) >> move;
  return !(result ^ x);
}
```
### 解析：
* 要求解读：题目的意思实际就是判断 x 能不用只用 n 位的二进制数表示。例如十进制的 5 需要用四位二进制数表示，即 ``0101``。需要注意的是最高位的0不能省略，因为否则表示 -2。
* 所以我们将 x 右移（32 - n）位然后移回，目的是为了将正确的补全符号位，然后与原x做比较，如果相同则表示x可以被n位的二进制补码所表示。

## divpwr2
### 要求
```
divpwr2 - Compute x/(2^n), for 0 <= n <= 30
Round toward zero
Examples: divpwr2(15,1) = 7, divpwr2(-33,4) = -2
Legal ops: ! ~ & ^ | + << >>
Max ops: 15
Rating: 2
```
### 函数
```C
int divpwr2(int x, int n) {
  int handle = (1 << n) + （～1 + 1）;
  int bias = (x >> 31) & handle;
  return (x + bias) >> n;
}
```
### 解析
* 整数除法当被除数为负数时商为向上舍入，故可以通过加入一个偏置来解决，利用公式为：「x/y」 = 「（x + y - 1) / y」
* ``bias``即为偏置的值，原理为利用``x >> 31``得到一个字，如果x是负数，这个字全为1，否则全为0.利用掩码handle屏蔽适当的位置，我们便可以得到期望的偏置值。

## negate
### 要求：
```
negate - return -x 
Example: negate(1) = -1.
Legal ops: ! ~ & ^ | + << >>
Max ops: 5
Rating: 2
```
### 函数:
```C
int negate(int x) {
  x = ~x;
  x = x + 0x1;
  return x;
}
```
### 解析：
求负数比较简单，根据补码的性质，取反加一即可。

## isPositive
### 要求：
```
isPositive - return 1 if x > 0, return 0 otherwise 
Example: isPositive(-1) = 0.
Legal ops: ! ~ & ^ | + << >>
Max ops: 8
Rating: 3
```
### 函数：
```C
int isPositive(int x) {
  return (~(x >> 31) & 0x1) ^ !x;
}
```
### 解析：
右移31位取反后与1进行按位与运算，负数得到0，正数和0得到1，此时0不符合要求，观察发现非零的非为0，0为1，故可以将所求数与x的非进行按位异或运算。  

## isLessOrEqual
### 要求：
```
isLessOrEqual - if x <= y  then return 1, else return 0 
Example: isLessOrEqual(4,5) = 1.
Legal ops: ! ~ & ^ | + << >>
Max ops: 24
Rating: 3
```
### 函数：
```C
int isLessOrEqual(int x, int y) {
  int handle = y + (~x + 1);
  int result = ~(handle >> 31) & 0x1;
  int i = x >> 31 & 1;
  int j = y >> 31 & 1;
  return (result & !(i ^ j)) | (!!(i - j + 1) & (i ^ j));
}
```
### 解析：
1. 最初博主的想法是用``y - x``判断所得结果的正负，但是出现的问题是会发生溢出，例如y为最大的整数，x为最小的负数，故此方法无法得出结果，但是这个方法给我们提供了一个分类讨论的思路。
2. 由最初的方法我们发现如何没有发生溢出那么结果是正确的否则是未知的，那么什么情况下发生了溢出什么情况下又没有呢？ 
  + 2-1 当 x y 同号时不会发生溢出（包含 x y 为零的情况），因此我们用 i j 取得 x y 的符号位并判断是否相同，相同时则直接利用resulte的结果。
  + 2-2 当 x y 异号时则返回的resulte是不准确的，故利用``!(i ^ j)``的返回值为零，屏蔽它对结果的影响。然后利用此时 x y 符号为的关系计算结果。应注意的是在第二种情况是应 & 一个``(i ^ j)``避免前一种情况可信但是返回值为零时，或运算得到第二种情况的结果。
3. 关于这道题的看法：减法运算出现溢出的情况后博主卡了好久，愣是没有想到分类讨论，所以在此劝告大家一定要放开思想，不要被自己局限到。datalab每道题的解法都有很多很多种，你总能找到一种的。

## ilog2
### 要求：
  ```
  ilog2 - return floor(log base 2 of x), where x > 0
  Example: ilog2(16) = 4
  Legal ops: ! ~ & ^ | + << >>
  Max ops: 90
  Rating: 4
  ```
### 函数：
  ```C
  int ilog2(int x) {
    int result = (!!(x >> 16)) << 4;
    result += (!!(x >> (8 + result)) << 3);
    result += (!!(x >> (4 + result)) << 2);
    result += (!!(x >> (2 + result)) << 1);
    result += (!!(x >> (1 + result)));
    return result;
  }
  ```
### 解析：
* 该题的本质就是求x的二进制表示中唯一的 1 在第几位。可以利用二分法的思想。
* 首先将x右移16位，然后判断所得结果是否为正数，若为正数，则 1 位于高十六位，将结果加16，否则位于低十六位，加0。应注意的是加法的实现方式。
* 然后重复上述该操作，注意的是此后的移动位数有所变化，与上次结果有关。最终可以确定 1 的位置。

## float_neg
### 要求：
```
float_neg - Return bit-level equivalent of expression -f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representations of
 *   single-precision floating point values.
 *   When argument is NaN, return argument.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 10
 *   Rating: 2
 ```
 ### 函数：
 ```C
 unsigned float_neg(unsigned uf){
  unsigned temp = uf & 0x7fffffff;
  unsigned resulte = uf ^ 0x80000000;
  if(temp > 0x7f800000){
    resulte = uf;
  }
  return resulte;
}
```
### 解析：
* 题目给出了一个32位的无符号数，要求我们把它看作一个单精度的浮点数的位级表示，并求出它的相反数。当该数字是NaN的时候将该数字返回。
* 利用异或将该数字的符号位取反，然后设置一个变量来判断uf是否为NaN，如果位NaN，将结果重新设置为uf。

## float_i2f
### 要求：
```
float_i2f - Return bit-level equivalent of expression (float) x
Result is returned as unsigned int, but
it is to be interpreted as the bit-level representation of a
single-precision floating point values.
Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
Max ops: 30
Rating: 4
```
### 函数：
```C
unsigned float_i2f(int x){
  unsigned sign = x >> 31 & 0x1;
  unsigned exp, fra, framask = 0x7fffff;
  int i, rounding;
  if(x == 0){
    return 0;
  }else if(x == 0x80000000){
    return 0xcf000000;
  }else{
    if(sign){
      x = -x;
    }
    i = 30;
    while(!(x >> i)){
      i--;
    }
    exp = i + 127;
    x = x << (31 - i);
    fra = (x >> 8) & framask;

    x = x & 0xff;
    rounding = x > 128 || ((x == 128) && fra & 1);
    fra += rounding;

    if(fra >> 23){
      fra &= framask;
      exp++;
    }
  }
  return (sign << 31) | (exp << 23) | fra;
}
```
### 解析
* 1-3行用来初始变量，``sign``表示符号位，``exp``表示阶码，``fra``表示尾码，``framask``表示尾码的掩码。
* 5-6行判断参数是否为零，是则直接返回。
* 7-8行判断是否为TMin，是则直接返回``0xcf00000``
* 10-12行判断是否为负数，若是负数，则求绝对值，因为整形负数的补码不好直接转换。
* 13-17求得第一个非零位，以及这是第几位，并加上相应的偏置，求的对应的阶码。
* 18行将x有效数据前的0清除，使其以有效数字开头。
* 19行确定初步的尾数部分（此时还没有舍入）。
* 21-27根据舍入规则，确定之前舍弃的八位是否应该进位，并进行相应操作。
* 最后根据浮点数的组成形式返回结果。

## float_twice
### 要求：
```
float_twice - Return bit-level equivalent of expression 2*f for
floating point argument f.
Both the argument and result are passed as unsigned int's, but
they are to be interpreted as the bit-level representation of
single-precision floating point values.
When argument is NaN, return argument
Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
Max ops: 30
Rating: 4
 ```
### 函数：
```C
unsigned float_twice(unsigned uf) {
  unsigned result = uf;
  if((result & 0x7f800000) == 0){
    result = ((result & 0x007fffff) << 1) | (result & 0x80000000);
  }else if((result & 0x7f800000) != 0x7f800000){
    result += 0x00800000;
  }
  return result;
}
```
### 解析：
* 求浮点数乘2的值，根据浮点数的三种情况分类讨论下即可。