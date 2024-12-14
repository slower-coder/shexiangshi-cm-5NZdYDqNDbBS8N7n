
题目：给你一个 32 位的有符号整数 x ，返回将 x 中的数字部分反转后的结果。如果反转后整数超过 32 位的有符号整数的范围 \[−231,  231 − 1] ，就返回 0。


**假设环境不允许存储 64 位整数（有符号或无符号）。**


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241213233410503-1418867673.png)


# ***01***、long类型字符串转换法


虽然题目要求不允许使用64位整数，但是我们还是先使用最简单最直观的最符合思维逻辑的方式实现，然后以此打开思维寻找出更好的方法。


看到此题我第一反应就是直接把整数x转为字符串，然后直接调用字符串方法反转字符串，最后把反转后的字符串转换为long类型，同时比较是否再有效范围内即可。


当然这是大致解题思路，其中还有许多细节需要处理。


首先需要处理负号问题，如果是负数我们需要取其绝对值，然后再反转绝对值，而在取绝对值时需要注意int的最小值int.MinValue为\-2147483648，而int.MaxValue最大值为2147483647，因此我们不能直接对int整数x直接取绝对值，而需要先把x转为long类型整数，不然会报错。


然后把绝对值反转成字符数组，同时判断正负号，如果是负数则需要在字符数组前加上负号’\-’。


最后直接使用long.TryParse方法对字符数组进行转换，同时判断其有效范围，并输出结果，具体代码如下：



```
//字符串long
public static int ReverseLongString(int x)
{
    //是否为负数
    var isNegative = x < 0;
    //取绝对值,必须要先转为long类型
    //否则int.MinValue -2147483648会报错
    var abs = Math.Abs((long)x);
    //把值转为字符串并反转，得到字符集合
    var reversedChars = abs.ToString().Reverse();
    if (isNegative)
    {
        //如果是负数则在字节数组前加入负号'-'
        reversedChars = reversedChars.Prepend('-');
    }
    //转换为long类型，并且是有效的int值，则返回结果
    if (long.TryParse(reversedChars.ToArray(), out long result) && result >= int.MinValue && result <= int.MaxValue)
    {
        return (int)result;
    }
    return 0;
}

```

# ***02***、int类型字符串转换法


既然题目中要求我们不能使用64位整数long类型，那么我们是否可以直接int类型进行转换呢？


根据上一个解法，同样的思路，我们先把int整数x转为字符串，然后直接使用int.TryParse进行转换即可。这样可以利用转换失败过滤掉所有溢出的值。


具体实现代码如下：



```
//字符串int
public static int ReverseIntString(int x)
{
    //把值转为字符串，并去掉负号'-'，最后反转，得到字符集合
    var reversed = x.ToString().TrimStart('-').Reverse();
    //转换为int，成功则返回
    if (int.TryParse(reversed.ToArray(), out int result))
    {
        //根据原始符号，返回结果
        return x < 0 ? -result : result;
    }
    return 0;
}

```

# ***03***、数学方法


上面两个方法本质还是通过字符串转换，就效率来说是比较低的，因此我们可以通过数学计算的方式来实现其转换。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241213233402276-706505312.jpg)


如上图，我们以把12345反转为例，详解讲解反转过程。


（1）通过12345%10，获取到尾数字5，而尾数字又将作为新数值的首数字，新数值5；


（2）通过1234%10，获取到尾数字4，新数值为5\*10\+4\=54；


（3）通过123%10，获取到尾数字3，新数值为54\*10\+3\=543；


（4）通过12%10，获取到尾数字2，新数值为543\*10\+2\=5432；


（5）通过1%10，获取到尾数字1，新数值为5432\*10\+1\=54321；


其中还有一个重点是关于溢出的判断，前两个方法本质都是通过转换方法触发异常来拦截溢出，而在此方法中我们可以在实时计算的过程中直接判断出是否溢出。


因为32位有符号整数x的取值范围是\-2147483648\<\=x\<\=2147483647，如果要保证反转过来不溢出，则在处理到第九位的时候整个值应该在（\-214748364，214748364）之间，不然结果肯定会溢出，而有效的int值首位数字最大为2，即使反转过来也不可能大于7或小于\-8，因此只需要判断第九位数字是否合法即可完成溢出判断。


具体实现代码如下：



```
//数学方法
public static int ReverseMath(int x)
{
    var result = 0;
    while (x != 0)
    {
        //判断溢出,因为输入的是32位的有符号整数 x
        //即输入的 -2147483648<=x<=2147483647
        //所以翻转后的最后一位是1或2并不会导致溢出
        //因此只需判断九位数 > int.MaxValue / 10 或者 < int.MinValue / 10
        if (result < int.MinValue / 10 || result > int.MaxValue / 10)
        {
            return 0;
        }
        //获取当前末尾的数字
        var digit = x % 10;
        //去掉末尾数字
        x /= 10;
        //反转并累积结果
        result = result * 10 + digit;
    }
    return result;
}

```

# ***04***、基准测试


我们做个简单的基准测试，分别对三种方法进行100万次随机生成整数值在范围\-2147483648至2147483647之间的值进行测试，得到如下结果。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241213233351803-1047272185.png)


通过上图不难发现数学方法在整体性能方面远远高于字符串处理方式。


***注***：测试方法代码以及示例源码都已经上传至代码库，有兴趣的可以看看。[https://gitee.com/hugogoos/Planner](https://github.com)


 本博客参考[MeoMiao 萌喵加速](https://biqumo.org)。转载请注明出处！
