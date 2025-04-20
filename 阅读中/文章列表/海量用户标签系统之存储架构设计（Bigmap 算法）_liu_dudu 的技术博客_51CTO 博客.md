---
source: https://blog.51cto.com/jackl/2797630
create: 2024-01-18 16:02
read: true
tags:
  - 系统架构
---

# 海量用户标签系统之存储架构设计（Bigmap 算法）_liu_dudu 的技术博客_51CTO 博客

## 1. 背景

我们在日常的工作中经常遇到这种场景

对一个用户添加许多的标签信息方便对用户身份进行搜索和精细化运营

ps: 本文我们不考虑用户身上的标签是怎么来的, 只讨论用户已经拥有标签的情况下怎么进行存储

需求分析

我们给用户做标签的目的是为了支持更加精细化的运营, 算是用户画像的一部分, 用户的标签来源可能跟消费, 登录, 浏览等记录都有关系

我们要做的是可以根据用户身上已经存在的标签, 筛选出来符合我们需求的用户

我们可以在大量的标签中查找具有某一些标签的用户, 或者获取某用户身上的所有标签

我们如果要满足以上的需求, 需要提供以下几个基本接口来方便进行数据查找

1.  查找某标签的所有用户以及非该标签的用户
2.  查找某个用户身上的所有标签
3.  判断某个用户是否有某个标签

一般来说对以上需求, 对于用户和用户身上的标签数据, 如果我们采用数据库来进行存储

可能会采用以下方式 (为了方便我们模拟了 7 个用户, 7 个标签, 以下测试都基于该假数据), 例如:

1.  使用字段标识标签信息

<table><thead><tr><th>id</th><th>name</th><th>vip</th><th>mobile</th><th>email</th><th>male</th><th>mac</th><th>supervip</th><th>lost</th></tr></thead><tbody><tr><td>1</td><td>小明</td><td>1</td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td></tr><tr><td>2</td><td>小花</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr><tr><td>3</td><td>小江</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td></tr><tr><td>4</td><td>小红</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr><tr><td>5</td><td>小九</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>0</td></tr><tr><td>6</td><td>小七</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>0</td></tr><tr><td>7</td><td>小四</td><td>1</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>1</td></tr></tbody></table>

或者是这样

1.  使用记录标识标签信息

<table><thead><tr><th>tag</th><th>uid</th><th>result</th></tr></thead><tbody><tr><td>vip</td><td>1</td><td>1</td></tr><tr><td>mobile</td><td>1</td><td>1</td></tr><tr><td>email</td><td>1</td><td>0</td></tr><tr><td>male</td><td>1</td><td>1</td></tr><tr><td>mac</td><td>1</td><td>0</td></tr><tr><td>supervip</td><td>1</td><td>1</td></tr><tr><td>lost</td><td>1</td><td>0</td></tr><tr><td>vip</td><td>2</td><td>0</td></tr><tr><td>mobile</td><td>2</td><td>1</td></tr><tr><td>email</td><td>2</td><td>0</td></tr><tr><td>male</td><td>2</td><td>0</td></tr></tbody></table>

以上两种方式功能上都可以达到我们想要的效果, 但第一种方式在标签数量非常多的时候明显是不合适的, 我们不可能给每个标签都添加一个字段, 那样性能和扩展性都损失非常大

在上面的两个表中第二个表相当于对第一个表进行了拆分, 增强了标签的扩展性. 如果我们采用第二种方式存储, 对于上面的需求 1,2,3 都能很好的满足

但是方式 2 依然有两个可能遇到的问题

1.  我们要查找在某一些标签的用户需要使用如下 sql

```
select
uid
from
tag_table
where
result = 1
and tag in ('vip','mobile','email','male','supervip','lost')
```

这样的语句在标签数万甚至数十万的时候对性能影响会非常大

1.  存储: 因为每个行记录同时标明了用户, 标签, 和结果, 所以其中的重复数据非常的多, 对数据库存储是个极大地浪费

Bitmap

## 2. Bitmap 的概念

Bitmap 翻译做中文称为 "位图", 其核心里面是充分利用一部分数据本身就存在的元属性 (空间 / 位置 / 容量) 信息, 我们这里主要是使用其中的每一位的位置信息, 达到使用一个信息表达两种含义的作用

其实就也是一种特殊的编码 (coding) 过程(或者叫多工(multiplex))

## 3. 什么是 bitmap 算法

1：Bit-Map 算法又名位图算法，其原理是，使用下标代替数值或特定的意义，使用这个位为 0 或者 1 代表特性是否存在。

2：Bit-Map 算法具有效率高，节省空间的特点，适用于对大量数据进行去重，查询等

## 4. 解决的问题

bitmap 可以用来有效解决两类问题

1.  存储大量值可以用布尔值标识的数据
2.  部分有用到交, 并, 差等集合运算的数据

第一个特性主要是利用位存储的节省空间的特性, 第二个是利用计算机位运算比较快速的特性

eg:

1.  以前的搜索引擎爬虫在处理网页爬取的时候需要给已经爬取过的网页做标记, 避免陷入死循环的重复爬取, 当时的搜索网站的爬虫就有一些采用过 bitmap 来给爬取过的网页做标记, 大致就是取页面的 url 取 hash, 然后处理成数字, 把对应的数字位置为 1
    
2.  微博里面你关注的 A 也关注了 B, 使用 B 的粉丝列表和你的关注列表进行交集运算就可以了, 同样 购买这件商品的人也购买了 M, 也可以用 购买这件商品的用户列表里面取某个用户购买过的某个商品即可

以上应用确实能有效的减少数据的存储容量和提高集合计算速度, 如果我们用这种方法来存储用户标签信息也能大量减少存储容量

但是怎么把用户标签的表信息数据转换成 bitmap 形式的数据呢?

## 5. 数据处理

我们如果要记录一个用户对应的一个标签的信息, 假如我们知道 5 号用户是小九, 而她是一位超级会员用户 (我们可以在上面的表中查到该信息)

我们要如何使用 bitmap 来表示这条信息呢

*   存储用户和标签的关系

我们可以这样:

1.  使用一个键`user:supervip`来记录所有用户是否是超级会员的信息, 这个值最初是空的字符串值, 表明没有超级会员用户
2.  我们为了标明 5 号用户是超级会员 可以使用这个键中对应位置的二进制位来表明会员的身份, 将这个键的第 5 位置为 1, 这样这个`user:supervip`值现在是’000001’(从第 0 位开始计算)
3.  同样, 如果`user:supervip`的值现在是’01001010’ 我们就可以知道 1,4,6 号用户都是超级会员用户

我们根据这个数据可以做到 2 点:

*   我们可以根据该标签数据键的对应位置的二进制位的值来判断以该位置为 id 的用户的标签结果
    
*   也可以查询某个标签下的所有用户

这样我们存储上万个标签也只需要上万个键

*   存储所有用户

但是我们如果需要查找不属于某个标签的用户怎么办啊, 如果直接对上一个例子取反肯定是不行的

为了解决这个问题我们需要一个存储所有用户的键

我们知道了所有用户, 知道了拥有某标签的用户

```
不含某标签的用户 = 总用户 - 含有某标签的用户
```

用二进制的操作方法就是使用`异或`, 举例:

我们有 7 个用户 (编号 1-7),5 号用户是超级 vip, 我们要查找所有不是 vip 的用户可以使用下面的运算

```
01111111 ^ 00001000 = 01110111 // 127 ^ 8 = 119
// 01111111:所有用户的二进制键 00001000:5号用户是超级会员的键 01110111:所有不是超级会员的用户
```

存储某用户的所有标签以上操作我们就能得到所有不是超级会员的用户

我们如果要获得用户的所有标签, 也可以将用户拥有的标签 id 在用户标签键中所对应的位置置为 1, 这样每一个用户的表示所有标签的键的最大位长度就是固定的, 比如:

我们可以用如下方式存储用户的所有标签

```
usertag:all:1 => 01101010 // 1号用户的所有标签
usertag:all:5 => 00010110 // 5号用户的所有标签
```

这样我们就能使用 bitmap 来满足以上基本查询需求

同样我们也可以将所有标签存储成一个`usertag:alltag`键, 再使用异或运算计算某用户不含有的标签

## 6. 实现方案

我们如果自己来对位运算做管理就有点麻烦了, 我们可以借助`redis`

`redis`原生提供了可以对字符串进行位操作的命令, 具体如下

```
$ SETBIT key pos value // 将 key 的第 pos 位设为 value(只能取1/0)
$ GETBIT key pos // 获取 key 的第 pos 位的值
$ BITOP cmd key1 key2 key3 ... // 对 key2,key3 等执行
$ BITOP AND destkey srckey1 srckey2 srckey3 ... srckeyN
$ BITOP OR destkey srckey1 srckey2 srckey3 ... srckeyN
$ BITOP XOR destkey srckey1 srckey2 srckey3 ... srckeyN
$ BITOP NOT destkey srckey
$ BITPOS key bit start end // 将 key 的 strat 到 end 位全部设为 bit(0/1)
$ BITCOUNT mykey 1 1
$ BITFIELD mystring SET i8 #0 100 i8 #1 200
```

我们就直接使用`redis`来存储数据了, 这样方便点

## 7. 预处理

我们这边为了方便直接使用 redis 提供的`setbit`,`getbit`和`bitop`来进行字符串的位操作

因为我们要存储用户标签, 所以我们首先需要对用户和标签进行编号, 这样我们需要两个表

用户表:

<table><thead><tr><th>uid</th><th>name</th></tr></thead><tbody><tr><td>1</td><td>小明</td></tr><tr><td>2</td><td>小花</td></tr><tr><td>3</td><td>小江</td></tr><tr><td>4</td><td>小红</td></tr><tr><td>5</td><td>小九</td></tr><tr><td>6</td><td>小七</td></tr><tr><td>7</td><td>小四</td></tr></tbody></table>

标签表:

<table><thead><tr><th>tid</th><th>name</th><th>备注</th></tr></thead><tbody><tr><td>1</td><td>vip</td><td>是否 vip</td></tr><tr><td>2</td><td>mobile</td><td>是否绑定手机</td></tr><tr><td>3</td><td>email</td><td>是否绑定邮箱</td></tr><tr><td>4</td><td>male</td><td>是否男性</td></tr><tr><td>5</td><td>mac</td><td>是否使用 Mac</td></tr><tr><td>6</td><td>supervip</td><td>是否年费会员</td></tr><tr><td>7</td><td>lost</td><td>是否易流失用户</td></tr></tbody></table>

## 8. 存储

我们这里为了性能考虑使用 redis 来进行存储

我们将最上面的表格数据转换成以下键值对

```
{
// 所有用户
"user:all":{
"01111111"
},
// 所有vip用户
"user:vip":{
"01001001"
},
// 所有绑定了手机的用户
"user:mobile":{
"01101010"
},
// 所有绑定了邮箱的用户
"user:email":{
"00001010"
},
// 所有男性用户
"user:male":{
"01010011"
},
// 用户1的所有标签
"usertag:all:1":{
"01101010"
},
//用户2的所有标签
"usertag:all:2":{
"00100001"
}
}
```

## 9. 查询操作

我们可以使用 redis 的命令`getbit`来查询某个键的某个位置的值  
比如, 我们要查询 5 号用户是否具有 vip 标签, 可以使用以下命令

```
getbit user:vip 5 // 返回 0
```

要查询某用户身上的所有标签可以使用如下

```
get usertag:all:1 // 获取用户1的所有标签 返回'01101010'
```

我们如果要获取某标签下的所有用户可以使用如下命令

```
get usertag:all:1 // 获取用户1的所有标签 返回'01101010'
```

查询不具有某个标签的用户

```
$ bitop xor user:not_vip user:all user:vip // 根据所有用户和具有标签的用户进行异或运算,得到不含有某标签的用户
$ get user:not_vip // 返回二进制字符串
```

我们现在知道如何快速的获取我们想要的数据了, 但是我们发现有时候我们获取到的都是二进制的数据例如 `00001000` 这种, 而群殴们想从这样的数据中获取的是 `[5]` 这样的比较易读的信息

我们需要有一个将二进制字符串 转化为对应位置为 1 的位置数组的形式

如: function(`01001010`) => `[1,4,6]`

## 10. 结果解析

这里我们提供两个函数来进行这样的操作

1.  遍历法

我们遍历二进制字符串中的每一位, 每遇到一个为 1 的位置就将该位置放入数组

这种方法比较慢, 不建议使用, 这里贴一个示例代码

```
def key2array(self, key): # 将二进制('\x05'->'0b00000101')变为数组[5,7], 表示第五位和第七位为1
tmpstr = ''.join([bin(i).replace('0b', '').zfill(8) for i in key])
arr = []
str_len = len(tmpstr)
for i in range(0, str_len):
if int(tmpstr[i]) == 1:
arr.append(i)
return (arr)
```

1.  查表

我们可以观察一下 redis 返回的二进制数据的特点, 每 8 个二进制位属于一个字节, 每个字节都可以表示成具体的数字 (如: 0,23,127) 这个数字最大也只能到 255, 而且同一个数字有可能出现非常多次, 而每个数字所对应的转换过后的位置数组都是固定的, 比如: 100(二进制: 1100100) => [1,2,5]

我们可以利用这一点, 提前制作一个 `0-255`的所对应的位置表, 然后每次处理 8 位, 处理完把当前处理的位数加上新表中对应的值就可以快速的得到这个值了

ps: 我们也可以扩大这个表的容量以提高速度

贴下示例代码:

```
def build_bit_table(self): # 生成0-255的表
arr = []
for i in range(0, 256):
tmp_arr = []
tstr = bin(i).replace('0b', '').zfill(8)
n = 0
for k in tstr:
n = n + 1
if int(k) == 1:
tmp_arr.append(n)
arr.append(tmp_arr)
self.bit_table = arr

def key2array(self, key): # 查表法
arr = []
n = 0
for i in key:
pos = self.bit_table[i]
for k in pos:
arr.append(n + k - 1)
n = n + 8
return (arr)
```

ps:jdk 中的 BitSet 就是对 bitmap 的一种简单实现

如果标签过于稀疏会不会浪费空间?

如果我们在一个很长的 bitmap 中只存除了极少量的数据是不是会对空间造成浪费呢?

例如: 在 bitmap 的第 40000 位置为 1, 那存储的数据大概就类似: 00000000000…0000000001

这样的数据前面的 39999 位都是 0, 不会浪费空间吗

## 11. Google 的 EWAHCompressedBitmap

Google 的 EWAHCompressedBitmap 就对这种情况做了优化

EWAHCompressedBitmap 将整个的二进制数据分成每 64 位一个的 word

一个空的 Bitmap 默认拥有 4 个 word 也就是 `4*64` 位

其中 word0 存储 bitmap 的头信息

当我们改变对应位置的比特位的值时 word 会跟着变化

当我们插入的值非常大的时候 (例如: 40000), 算法会根据当前的值 创建两个新的 word

一个用于存储第 40000 个数据所在的 word 的信息 (LW), 还有一个存储跨度信息 (称为: 跨度 word /RLW )

假如说我们给一个空的 bitmap, 我们插入 40000 的话正常情况下会有 6 个 word, 前 4 个是头信息 word+3 个空 word, 第 6 个中保存 40000 这个数字所在的位置信息, 第 5 个 word 中保存从第 4-625 word 的跨度信息, 第 626word 中存储有 40000 这个数据

ps: 第一个 word 存储头信息, 625 = floor((40000 + 1) / 64 )

存储跨度信息的 word 和普通的存储数据的 word 虽然空间一样但是存储的内容不一样, 存储跨度信息的 word 大概内容这样

```
前32位存储 `当前跨度word(RLW)横跨了多少空word`    
后32位存储 `当前跨度word(RLW)后方有多少个连续的LW`
```

当我们存储 位置在跨度 word(RLW) 之中的数据 (例如: 20000), RLW 会进行分裂

变成 3 个 word, 中间一个存储 20000 所在的 LW 信息, 前后各有一个 RLW 保存新的跨度信息

EWAHCompressedBitmap 对应的 maven 依赖如下：

*   ```
    com.googlecode.javaewahJavaEWAH1.1.0
    ```