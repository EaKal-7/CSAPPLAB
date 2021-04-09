# Bomblab

听Q师傅说这个是re的入门，那我是真的菜，汇编是一边学一边看的，主要是一般都直接ida反编译，都没怎么重视汇编，这次一定好好学（下次一定

### 1

![image-20210407234313042](https://raw.githubusercontent.com/EaKal-7/Images/main/7ymn3lATg5j2oFk.png)

对于call 0x401338这个地址我们可以猜测大概是一个字符串比较，然后他将0x402400这个位置的东西赋值给了esi，不难想象应该是比较输入输出然后看.

IDA特供

```c
__int64 __fastcall strings_not_equal(_BYTE *a1, _BYTE *a2)
{
  _BYTE *v2; // rbx
  _BYTE *v3; // rbp
  int v4; // er12
  int v5; // eax
  unsigned int v6; // edx

  v2 = a1;
  v3 = a2;
  v4 = string_length(a1);
  v5 = string_length(a2);
  v6 = 1;
  if ( v4 == v5 )
  {
    if ( *a1 )
    {
      if ( *a1 == *a2 )
      {
        do
        {
          ++v2;
          ++v3;
          if ( !*v2 )
            return 0;
        }
        while ( *v2 == *v3 );
        v6 = 1;
      }
      else
      {
        v6 = 1;
      }
    }
    else
    {
      v6 = 0;
    }
  }
  return v6;
}
```



显然,跳过第一个bomb就是输出那一串字符。

（这里我尝试了一下让*a1=0，也就是不申请a1的地址，但是a1是输入，貌似异想天开了，所以就只能是a1==a2了）

answer:	Border relations with Canada have never been better.

### 2

![image-20210408170714090](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210408170714090.png)

![image-20210408171244346](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210408171244346.png)

先用IDA写吧，以后看得懂一点汇编了再说一下汇编，这里从IDA可以直接看到，它是先读取了六个数字，然后内部就不截图了，v3就是第一个数的值，这里发现是要1，然后看到后面的逻辑，后面每个数要是前面的两倍，显而易见就是1 2 4 8 16 32

（PS:Q师傅别骂了,再骂人没了

### 3

![image-20210408175459815](https://raw.githubusercontent.com/EaKal-7/Images/main/ksqPud2T7myWE6O.png)

这里需要注意的是![image-20210409162419240](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210409162419240.png)

这里v8这个数组被定义成了int64,这时候我们看下面的代码中v8[0]被拆成了两部分运行，这里就是用ida的不好了，ida在反编译时会犯病，查看gdb

![image-20210409170100591](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210409170100591.png)

可以发现读的并不是int64 可以发现读取的应该是

![image-20210409170414928](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210409170414928.png)

所以在ida里面把v8数组取成int（我这里错了好久，我以为是取v8[0]的高八位和低八位，别骂了，再也不用ida了（下次还敢

那接下来就很简单了

就是输入第一个数然后在反编译里找对应的。

Answer:	

0 207

1 311

2 707

3 256

4 389

5 206

6 682

7 327

### 4



![image-20210409172842957](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210409172842957.png)

这题跟上题一样犯病，已经改过来了。

然后先看判断条件，读取数字不能等于2，还有第一个数字要大于14



![image-20210409173028230](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210409173028230.png)

![image-20210409173041484](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210409173041484.png)

这里应该是result返回0,v8[1]返回0.然后看向func4这个函数，里面自己有一个循环

其实这个函数给我的感觉有点像二叉树，但是又不是，反正很绕，但是显然要返回result只有两个办法，一种是直接让a1=v3=7，一种是在两个if里面套出result=0，但是显然要想让两个if出0显然不可能，因为两个return会不断回跳，直到v3=a1，但是第二个return是有一个+1的，所以不能经过v3<a1的情况。为了避免麻烦还是直接让a1=v3=7吧。

(这题讲真，有ida的话有手就行，gdb搞了我半天，我没搞出来，那个jmp没看懂，原来是递归，我裂了)

so ,The Answer :7 0

### 5

![image-20210410004452131](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210410004452131.png)

这题被小坑了一下，就在那个v3取值那里，我以为是int化，结果要的是ascii码，我裂了

![image-20210410004647615](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210410004647615.png)

反正就是输入六个字母，使之acsii的低四位的表达在数组里能变成出flyers；

0x9FE567 (数组需要的低四位)

Answer:	yonuvw（这个答案很多，就是低四位符合就行，又是一道有手就行的题。

### 6

源码相当长，又臭又长，（TMD写了两个小时不断地在ida和gdb两边跳，汇编看不懂太痛苦了，ida显示又有问题

一步一步来

![image-20210410014112722](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210410014112722.png)

先是读入六个数字，然后一个死循环，这里直接说意思，就是必须要出六个数，这六个数小于等于6，且互不相同。

这里ida有问题，他这里是>，但是在gdb里面

```
0x0000000000401121 <+45>:    jbe    0x401128 <phase_6+52>
```

用的是jbe所以是小于等于。

![image-20210410014348288](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210410014348288.png)

这里是与7取互补，以上都是这道题最简单的地方。

![image-20210410014423255](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210410014423255.png)

这里开始变态起来了，这里看了半天看不懂这个node1是个der，联想到这样的地址操作还有node这个名字，猜测大概是链表，而且在ida里，这个nodeTMD是个UNKNOWN，这个v6=v6[1]的这个操作说实话，我现在还不是很懂，为什么ida会读成这个样子，希望有师傅看到这里能帮我答题解惑，然后我就看了眼gdb

![image-20210410014836946](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210410014836946.png)

才明白这个链表中的node早就在那里了，甚至整个链表中的值已经在寄存器了，我们决定的只是这个链表的顺序，

![image-20210410015132918](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210410015132918.png)

后面这一串大概是按照我们的顺序构建新链表吧（大概，我也是猜的，因为我也不是看的很懂，到时候汇编懂一点，去看汇编，这个ida是有点难用。

PS:睡了两个小时，我发现上面那一步已经把节点值取出来了，现在又搞不懂这个for有个der用了，现在把意思重新表达一下，其实这道题只是把一个链表中的值拿出来，并且从大到小。

然后下面那个部分应该是判别是否在整个链表中递减，然后我把node里的东西x出来了

![image-20210410014004232](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210410014004232.png)

14c	a8	39c	2b3	1dd	1bb	

原顺序： 3 4 5 6 1 2

取补后：4 3 2 1 6 5

![image-20210410021726880](https://raw.githubusercontent.com/EaKal-7/Images/main/image-20210410021726880.png)

bomblab done，学到的东西还是挺多的，学会了怎么用gdb，稍微看一点点的汇编（汇编真的要认真学啊