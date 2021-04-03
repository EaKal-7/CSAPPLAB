# csapp data lab

### 1
```c
/*

 * bitXor - x^y using only ~ and & 

 * Example: bitXor(4, 5) = 1

 * Legal ops: ~ &

 * Max ops: 14

 * Rating: 1
*/
   int bitXor(int x, int y) {
     return ~(~(x&~y)&~(~x&y));
   }
```
   这个学过一点离散数学都应该知道，嘻嘻。其实我一开始写出来的表达式 "(~x&y)|(x&~y)"把这个化一下就行了。

### 2

```c
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
  return 1<<31;

}
```
//这题的意思是要输出最小的二进制补码,就直接输出就好了
### 3
```c
/* isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
  int y=x+1;
  x+=y;
  x=~x;
  y=!y;//4
  x+=y;//5
  return !x;
}
```
这题参考了CSDN，确实没想到，方法还是比较奇妙的。首先说明以下这题的意思，如果x为最大的二进制补码就输出1，否则为0；
这题不难想到最大的补码应该是符号位为0，其余都是1。因为输出是0or1，所以考虑到将x化为全0，然后做！运算，这样得到的就是1or0，其次要排除-1，即符号位为1，其余也都是1，这里用4,5行做了排除。
### 4
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
  int BBits=0xAA;
  BBits=BBits+(BBits<<8);
  BBits=BBits+(BBits<<16);
  return (x&BBits)^BBits;
}
```
这题目说实话贼简单，先说意思吧，就是如果一个数的奇数位全为1就输出1，只要简单的构造一个掩码，这个掩码为最大的奇数全为1的数，然后做异或。（这里插一个题外话，这个lab的构造的常数只能0-255，所以，不能直接创建0XAAAAAAAAAAA
### 5
```c
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return ~x+1;
}
```
这题目就是不用·，然后输出·x，也是有手⑨☆。就是常识题。

### 6

```c
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
  int sigh=x>>31;
  int up=0x39;
  int down=0x30;
  down =~down+1;
  up=(up+~x+1)>>31;
  down=(down+x)>>31;
  return !(sigh|up|down);
}
```

这题比较困难建议自己看着理解一下，一开始是打算做每一位运算的，但是发现这个方法只适用0~9但是没有普遍性，所以写了一个具有普遍性的。

### 7

```c
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
  x=!!x;
  x=~x+1;
  int result=(x&y)+(~x&z);
  return result;
}
```

这题就是~0+1和~1+1的技巧，貌似也没啥。

### 8

```c
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  int sigh=0x1<<31;
  int bitXor=(!(x&sigh))^(!(y&sigh));

  int m=~x+1+y;
  m=m>>31;
  return ((!bitXor)&(!m)|(bitXor&(x>>31)));
}
```

略

### 9

```c
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int logicalNeg(int x) {
  return ((x|(~x+1))>>31)+1;
}
```

这里需要注意的是，补码右移的性质

负数在内存中表示的话最高位是符号位 负数为 1

做右移的话最高位会用1填充。

右移31位的话那么最后的数就是 0xffffffff了。
这个在内存中表示的就是有符号数的 -1

其次就是0的负数最高位仍然是0。

### 10

```c
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x) {
  int i16,i8,i4,i2,i1,i0;
  int sigh=x>>31;
  x=(sigh&~x)|(~sigh&x);

  i16=!!(x>>16)<<4;
  x=x>>i16;
  i8=!!(x>>8)<<3;
  x=x>>i8;
  i4=!!(x>>4)<<2;
  x=x>>i4;
  i2=!!(x>>2)<<1;
  x=x>>i2;
  i1=!!(x>>1);
  x=x>>i1;
  i0=x;
  return i16+i8+i4+i2+i1+i0+1;
}
```

就是让你判断x用补码表示最少需要几位

就是一个二分的思想。

### 11

````c
/* 
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatScale2(unsigned uf) {
  int exp=(uf&0x7f800000)>>23;
  int sigh=uf&(1<<31);
  if(exp==0)  return uf<<1|sigh;
  if(exp==255)  return uf;
  exp+=1;
  if(exp==255)  return 0x7f800000|sigh;
  return (exp<<23)|(uf&0x807fffff);
}
````

求2乘一个浮点数

首先需要对浮点数的存放有一个初步认识，浮点数的32位中分了三块，这个自行csdn

这里需要对无穷大，0特殊处理，（其实看了答案还有NaN，和无穷小，虽然我不知道NaN是怎么寄存的，但是问题不大

### 12

```c
/* 
 * floatFloat2Int - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
int floatFloat2Int(unsigned uf) {
  int sigh=uf>>31;
  int exp=((uf&0x7f800000)>>23)-127;
  int result=(uf&0x007fffff)|0x00800000;
  if(!(uf&0x7fffffff)) return 0;
  if(exp> 31) return 0x80000000;
  if(exp<0) return 0;
}
```

略