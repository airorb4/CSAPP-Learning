#### bitXor

需要使用与运算和非运算实现一个异或函数。用数电和离散的知识可解。

```c
//1
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
  return ~(~(~x & y) & ~(x & ~y));
//   return 2;
}

$ ./dlc -e ./bits.c
dlc:./bits.c:227:bitXor: 8 operators
...

$ ./btest -f bitXor
Score	Rating	Errors	Function
 1	1	0	bitXor
Total points: 1/1
```

#### tmin

返回 int 最小值的二进制补码，即 `0x80000000`，那么将 1 左移 31 位即可。

```c
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
  int a = 1;
  return (a << 31);
//   return 2;

}

$ ./dlc -e ./bits.c
...
dlc:./bits.c:239:tmin: 1 operators
...

$ ./btest -f tmin
Score	Rating	Errors	Function
 1	1	0	tmin
Total points: 1/1
```

#### isTmax

如果 x 为 int 的正最大值，即 `0x7fffffff`，则返回 1；否则返回 0。

没有移位..~~上网抄~~

```c
//2
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
  int i = x + 1;
  x = x + i;
  x = ~x;
  i = !i; // check overflow
  x = x + i;
  return !x;

  // int y = 0 ;
  //   return !(x^y);
//   return 2;
}

$ ./dlc -e ./bits.c
...
dlc:./bits.c:259:isTmax: 6 operators
...

$ ./btest -f isTmax
Score	Rating	Errors	Function
 1	1	0	isTmax
Total points: 1/1
```

当传入的 x 为 `0x7fffffff` 时：
1. `i = 0x80000000 = 0x7fffffff + 1;`
2. `x = 0xffffffff = 0x7fffffff + 0x80000000;`
3. `x = 0 = ~0xffffffff;`

考虑到当传入的 x 为 `0xffffffff` 时也会在第三步有 x == 0，则应设置检查将两种情况加以区分：

4. `i = 0 = !0x80000000; // check overflow: 当传入的 x 为 0xffffffff 时，这里的 i 为 1`
5. `x = 0 = 0 + 0;`
6. `return !0;`

即可实现如果 x 为 int 的正最大值时返回 1；否则返回 0。

#### allOddBits

如果奇数位全为 1，则返回1；否则返回 0。

当一个 32 位宽（int）的二进制数的奇数位全为 1 时，其值为 `0xAAAAAAAA`。将一个数 x 与 `0xAAAAAAAA` 相与，返回的结果相当于一个将 x 的偶数位清零，而将 x 的奇数位上的值保留的数。

在本题中，将输入 x 与 `0xAAAAAAAA` 进行与运算，可取得 x 中所有奇数位上的值。之后将得数与 `0xAAAAAAAA` 相异或，因为一个数与自身相异或的结果为 0，所以如果 x 中奇数位全为 1 的话，它与 `0xAAAAAAAA` 相异或的结果应为 0，如此便实现了必要的检验。

```c
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x) {
  int a = 0xaa;
  a = (a << 8) | a; // 移位构造 0xAAAAAAAA, 作为检验标准
  a = (a << 8) | a;
  a = (a << 8) | a;
  a = (a << 8) | a;
    return !((x & a) ^ a);
//   return 2;
}

$ ./dlc -e ./bits.c
...
dlc:./bits.c:276:allOddBits: 11 operators
...

$ ./btest -f allOddBits
Score	Rating	Errors	Function
 2	2	0	allOddBits
Total points: 2/2
```

#### negate

返回相反数。本质上是求补码，所以只要按位取反再加 1 即可。

```c
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
    x = ~x + 1;
  return x;
}

$ ./dlc -e ./bits.c
...
dlc:./bits.c:287:negate: 2 operators
...

$ ./btest -f negate
Score	Rating	Errors	Function
 2	2	0	negate
Total points: 2/2
```

#### isAsciiDigit 

判断输入是否位于指定范围内，总的来说就是要判断 `(x - 0x30) >= 0 && (0x39 - x) <= 0` 的值。这里只有加号，没有减号，不过可以用补码表示负数，再和输入 x 相加，以此来用加号替代减号。

```c
//3
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x) {
    int sign = 1 << 31;
    /* 取上下界的符号 */
    int upper_bound = (x + ~0x30 + 1) & sign;
    int lower_bound = (0x39 + ~x + 1) & sign;
    return !(upper_bound | lower_bound);
//   return 2;
}
```

#### conditional

感觉有点难..这里用 f (或 of) 跟 y (或 z) 进行异或，其中 f (或 of) 的值只会为 `0` 或 `~0`，可以得到 y (或 z) 原本的值或 y (或 z) 的反码。之后再和 of (或 f) 相与，可得到 y (或 z) 原本的值或 0。总的来说起到了条件选择的作用。

```c
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
    int f = ~(!x) + 1;
    int of = ~f;
    return ((f ^ y) & of) | ((of ^ z) & f);
//   return 2;
}

$ ./dlc -e ./bits.c
...
dlc:./bits.c:318:conditional: 9 operators
...

$ ./btest -f conditional
Score	Rating	Errors	Function
 3	3	0	conditional
Total points: 3/3