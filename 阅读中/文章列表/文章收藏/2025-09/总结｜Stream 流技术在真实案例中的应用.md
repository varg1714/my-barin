---
source: https://mp.weixin.qq.com/s/mZbejfgxLVwO7pu4nkRjIg
create: 2024-08-16 16:30
read: false
---

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naL7zVHZH429Po1HLpbichP9SLVicPtoxkI2WhMxUibFwG9U1dUOJu33R5KD9ib25hmibaaZLLldnh8dA4A/640?wx_fmt=jpeg&from=appmsg)

阿里妹导读

本文不仅提供了理论上的讲解，还通过实际代码示例展示了如何应用 Stream API 来解决常见的编程问题。

在日常开发中，有很多对象转化、链表去重、分批次服务调用等场景，这些场景用 for 循环或者 if-else 实现会让代码冗长、容易出错且效率不高。在查看项目代码的过程中以及师兄的引导下，学到了很多新的使用方式。对此，将 Stream 流进行一次整理。

案例引入

在 JAVA 中，涉及到对数组、Collection 等集合类中的元素进行操作的时候，通常会通过循环的方式进行逐个处理，或者使用 Stream 的方式进行处理。

假设我们遇到了这么一个需求：从给定句子中返回单词长度大于 5 的单词列表，按长度倒序输出，最多返回 3 个。

在学校里面，或者在未接触 Stream 流的时候，我们可能会这样写函数：

```
public List<String> sortGetTop3LongWords(@NotNull String sentence) {
    // 先切割句子，获取具体的单词信息
    String[] words = sentence.split(" ");
    List<String> wordList = new ArrayList<>();
    // 循环判断单词的长度，先过滤出符合长度要求的单词
    for (String word : words) {
        if (word.length() > 5) {
            wordList.add(word);
        }
    }
    // 对符合条件的列表按照长度进行排序
    wordList.sort((o1, o2) -> o2.length() - o1.length());
    // 判断list结果长度，如果大于3则截取前三个数据的子list返回
    if (wordList.size() > 3) {
        wordList = wordList.subList(0, 3);
    }
    return wordList;
}
```

然而，如果我们用上了 Stream 流：

```
public List<String> sortGetTop3LongWordsByStream(@NotNull String sentence) {
    return Arrays.stream(sentence.split(" "))
            .filter(word -> word.length() > 5)
            .sorted((o1, o2) -> o2.length() - o1.length())
            .limit(3)
            .collect(Collectors.toList());
}
```

可以直观得看出，代码缩短了几乎一半。所以，了解并更好的使用 Stream 流能够让我们的代码简洁明了，一气呵成。

一、Stream 流初步了解

概括讲，可以将 Stream 流操作分为 3 种类型：

*   创建 Stream
    
*   Stream 中间处理
    
*   终止 Steam

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuNdQbh70SMulZ27Zk7DcvcAIXJCDIDpPiavrtXJ60adJydia3jp1mbGsQ/640?wx_fmt=other&from=appmsg)

每个 Stream 管道操作都包含若干方法，先列举一下各个 API 的方法：

**1.1 开始管道**

主要负责新建一个 Stream 流，或者基于现有的数组、List、Set、Map 等集合类型对象创建出新的 Stream 流。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuhKiaETQpuKJVicHCMy0wXhE3gVleiaDibiayib0j8ItycmKczdQ7Pe5VU2Ow/640?wx_fmt=png&from=appmsg)

**1.2 中间管道**

负责对 Stream 进行处理操作，并返回一个新的 Stream 对象，中间管道操作可以进行叠加。

<table width="-149"><colgroup><col width="325"><col width="501"></colgroup><tbody><tr data-cangjie-key="117" data-sticky="false"><td data-cangjie-key="119" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="61"><p><span>API</span></p></td><td data-cangjie-key="124" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="1139"><p><span>功能说明</span></p></td></tr><tr data-cangjie-key="129" data-sticky="false"><td data-cangjie-key="131" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="61"><p><span>filter()</span></p></td><td data-cangjie-key="136" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="1139"><p><span>按照条件过滤符合要求的元素， 返回新的 stream 流。</span></p></td></tr><tr data-cangjie-key="141" data-sticky="false"><td data-cangjie-key="143" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="61"><p><span>map()</span></p></td><td data-cangjie-key="148" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="1139"><p><span>将已有元素转换为另一个对象类型，一对一逻辑，返回新的 stream 流。</span></p></td></tr><tr data-cangjie-key="153" data-sticky="false"><td data-cangjie-key="155" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="61"><p><span>flatMap()</span></p></td><td data-cangjie-key="160" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="1139"><p><span>将已有元素转换为另一个对象类型，一对多逻辑，即原来一个元素对象可能会转换为 1 个或者多个新类型的元素，返回新的 stream 流。</span></p></td></tr><tr data-cangjie-key="165" data-sticky="false"><td data-cangjie-key="167" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="61"><p><span>limit()</span></p></td><td data-cangjie-key="172" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="1139"><p><span>仅保留集合前面指定个数的元素，返回新的 stream 流。</span></p></td></tr><tr data-cangjie-key="177" data-sticky="false"><td data-cangjie-key="179" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="61"><p><span>skip()</span></p></td><td data-cangjie-key="184" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="1139"><p><span>跳过集合前面指定个数的元素，返回新的 stream 流。</span></p></td></tr><tr data-cangjie-key="189" data-sticky="false"><td data-cangjie-key="191" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="61"><p><span>concat()</span></p></td><td data-cangjie-key="196" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="1139"><p><span>将两个流的数据合并起来为 1 个新的流，返回新的 stream 流。</span></p></td></tr><tr data-cangjie-key="201" data-sticky="false"><td data-cangjie-key="203" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="61"><p><span>distinct()</span></p></td><td data-cangjie-key="208" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="1139"><p><span>对 Stream 中所有元素进行去重，返回新的 stream 流。</span></p></td></tr><tr data-cangjie-key="213" data-sticky="false"><td data-cangjie-key="215" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="61"><p><span>sorted()</span></p></td><td data-cangjie-key="220" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="1139"><p><span>对 stream 中所有的元素按照指定规则进行排序，返回新的 stream 流。</span></p></td></tr><tr data-cangjie-key="225" data-sticky="false"><td data-cangjie-key="227" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="61"><p><span>peek()</span></p></td><td data-cangjie-key="232" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="1139"><p><span>对 stream 流中的每个元素进行逐个遍历处理，返回处理后的 stream 流。</span></p></td></tr></tbody></table>

**1.3 终止管道**

顾名思义，通过终止管道操作之后，Stream 流将会结束，最后可能会执行某些逻辑处理，或者是按照要求返回某些执行后的结果数据。

<table width="-162"><colgroup><col width="325"><col width="506"></colgroup><tbody><tr data-cangjie-key="245" data-sticky="false"><td data-cangjie-key="247" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>API</span></p></td><td data-cangjie-key="252" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>功能说明</span></p></td></tr><tr data-cangjie-key="257" data-sticky="false"><td data-cangjie-key="259" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>count()</span></p></td><td data-cangjie-key="264" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>返回 stream 处理后最终的元素个数。</span></p></td></tr><tr data-cangjie-key="269" data-sticky="false"><td data-cangjie-key="271" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>max()</span></p></td><td data-cangjie-key="276" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>返回 stream 处理后的元素最大值。</span></p></td></tr><tr data-cangjie-key="281" data-sticky="false"><td data-cangjie-key="283" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>min()</span></p></td><td data-cangjie-key="288" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>返回 stream 处理后的元素最小值。</span></p></td></tr><tr data-cangjie-key="293" data-sticky="false"><td data-cangjie-key="295" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>findFirst()</span></p></td><td data-cangjie-key="300" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>找到第一个符合条件的元素时则终止流处理。</span></p></td></tr><tr data-cangjie-key="305" data-sticky="false"><td data-cangjie-key="307" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>findAny()</span></p></td><td data-cangjie-key="312" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>找到任何一个符合条件的元素时则退出流处理，这个对于串行流时与 findFirst 相同，对于并行流时比较高效，任何分片中找到都会终止后续计算逻辑。</span></p></td></tr><tr data-cangjie-key="317" data-sticky="false"><td data-cangjie-key="319" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>anyMatch()</span></p></td><td data-cangjie-key="324" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>返回一个 boolean 值，类似于 isContains(), 用于判断是否有符合条件的元素。</span></p></td></tr><tr data-cangjie-key="329" data-sticky="false"><td data-cangjie-key="331" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>allMatch()</span></p></td><td data-cangjie-key="336" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>返回一个 boolean 值，用于判断是否所有元素都符合条件。</span></p></td></tr><tr data-cangjie-key="341" data-sticky="false"><td data-cangjie-key="343" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>noneMatch()</span></p></td><td data-cangjie-key="348" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>返回一个 boolean 值， 用于判断是否所有元素都不符合条件。</span></p></td></tr><tr data-cangjie-key="353" data-sticky="false"><td data-cangjie-key="355" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>collect()</span></p></td><td data-cangjie-key="360" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>将流转换为指定的类型，通过 Collectors 进行指定。</span></p></td></tr><tr data-cangjie-key="365" data-sticky="false"><td data-cangjie-key="367" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>toArray()</span></p></td><td data-cangjie-key="372" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>将流转换为数组。</span></p></td></tr><tr data-cangjie-key="377" data-sticky="false"><td data-cangjie-key="379" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>iterator()</span></p></td><td data-cangjie-key="384" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>将流转换为 Iterator 对象。</span></p></td></tr><tr data-cangjie-key="389" data-sticky="false"><td data-cangjie-key="391" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="38"><p><span>foreach()</span></p></td><td data-cangjie-key="396" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" width="970"><p><span>无返回值，对元素进行逐个遍历，然后执行给定的处理逻辑。</span></p></td></tr></tbody></table>

二、Stream 方法的使用

**2.1map 与 flatMap**

在项目中，经常看到也经常使用到 map 与 flatMap，比如代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuApoUNxnSrT2fOA0pEdIDA2G9OGhFs0kB7xeNjFgsaOibkbPJ0oB5CzQ/640?wx_fmt=other&from=appmsg)

那两者有什么相同和不同呢？

map 与 flatMap 都是用于转换已有的元素为其它元素，区别点在于：

*   map 必须是一对一的，即每个元素都只能转换为 1 个新的元素；
    
*   flatMap 可以是一对多的，即每个元素都可以转换为 1 个或者多个新的元素；

下面两张图形象地说明了两者之间的区别：

map 图：

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7Cu0oOQ64PyW69UJOgfJs6PQS8WF59Hs6uibfhpFo6AdrwPQYqkBGHutTA/640?wx_fmt=other&from=appmsg)

flatMap 图：

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuneZ6nvwmDMhtD29lbP0o3AQOvQInia8UgNyzf4uuHbiaPicdlRJyUShgg/640?wx_fmt=other&from=appmsg)

## map 用例：

有一个字符串 ID 列表，现在需要将其转为别的对象列表。

```
/**
 * map的用途：一换一
 */
List<String> ids = Arrays.asList("205", "105", "308", "469", "627", "193", "111");
        // 使用流操作
List<NormalOfferModel> results = ids.stream()
        .map(id -> {
            NormalOfferModel model = new NormalOfferModel();
            model.setCate1LevelId(id);
            return model;
        })
        .collect(Collectors.toList());
System.out.println(results);
```

执行之后，会发现每一个元素都被转换为对应新的元素，但是前后总元素个数是一致的：  

```
[
NormalOfferModel{cate1LevelId='205', imgUrl='null', linkUrl='null', ohterurl='null'},
NormalOfferModel{cate1LevelId='105', imgUrl='null', linkUrl='null', ohterurl='null'},
NormalOfferModel{cate1LevelId='308', imgUrl='null', linkUrl='null', ohterurl='null'}, 
NormalOfferModel{cate1LevelId='469', imgUrl='null', linkUrl='null', ohterurl='null'}, 
NormalOfferModel{cate1LevelId='627', imgUrl='null', linkUrl='null', ohterurl='null'},
NormalOfferModel{cate1LevelId='193', imgUrl='null', linkUrl='null', ohterurl='null'}, 
NormalOfferModel{cate1LevelId='111', imgUrl='null', linkUrl='null', ohterurl='null'}
    ]
```

## flatMap 用例：

现有一个句子列表，需要将句子中每个单词都提取出来得到一个所有单词列表：

```
List<String> sentences = Arrays.asList("hello world","Hello Price Info The First Version");
// 使用流操作
List<String> results2 = sentences.stream()
        .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
        .collect(Collectors.toList());
System.out.println(results2);
```

结果：

```
[hello, world, Hello, Price, Info, The, First, Version]
```

这里需要补充一句，flatMap 操作的时候其实是先每个元素处理并返回一个新的 Stream，然后将多个 Stream 展开合并为了一个完整的新的 Stream，如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuibsbJHARWpv9FmdMVicicx0MUHV1IfsdPxFH2Cgic2qezQyibfmM2DYicDwA/640?wx_fmt=other&from=appmsg)

**2.2 peek 和 foreach 方法**

peek 和 foreach，都可以用于对元素进行遍历然后逐个处理。

但根据前面的介绍，peek 属于中间方法，而 foreach 属于终止方法。这也就意味着 peek 只能作为管道中途的一个处理步骤，而没法直接执行得到结果，其后面必须还要有其它终止操作的时候才会被执行；而 foreach 作为无返回值的终止方法，则可以直接执行相关操作。

```
public void testPeekAndforeach() {
    List<String> sentences = Arrays.asList("hello world","Jia Gou Wu Dao");
    // 演示点1：仅peek操作，最终不会执行
    System.out.println("----before peek----");
    sentences.stream().peek(sentence -> System.out.println(sentence));
    System.out.println("----after peek----");
    // 演示点2：仅foreach操作，最终会执行
    System.out.println("----before foreach----");
    sentences.stream().forEach(sentence -> System.out.println(sentence));
    System.out.println("----after foreach----");
    // 演示点3：peek操作后面增加终止操作，peek会执行
    System.out.println("----before peek and count----");
    sentences.stream().peek(sentence -> System.out.println(sentence)).count();
    System.out.println("----after peek and count----");
}
```

输出结果可以看出，peek 独自调用时并没有被执行、但 peek 后面加上终止操作之后便可以被执行，而 foreach 可以直接被执行：

```
----before peek----
----after peek----
----before foreach----
hello world
Jia Gou Wu Dao
----after foreach----
----before peek and count----
hello world
Jia Gou Wu Dao
----after peek and count----
```

**2.3 filter、sorted、distinct、limit**

这几个都是常用的 Stream 的中间操作方法，具体的方法的含义在上面的表格里面有说明。具体使用的时候，可以根据需要选择一个或者多个进行组合使用，或者同时使用多个相同方法的组合：

```
public void testGetTargetUsers() {
    List<String> ids = Arrays.asList("205","10","308","49","627","193","111", "193");
    // 使用流操作
    List<OfferModel> results = ids.stream()
            .filter(s -> s.length() > 2)
            .distinct()
            .map(Integer::valueOf)
            .sorted(Comparator.comparingInt(o -> o))
            .limit(3)
            .map(id -> new OfferModel(id))
            .collect(Collectors.toList());
    System.out.println(results);
}
```

上面的代码片段的处理逻辑:

```
1.使用filter过滤掉不符合条件的数据
2.通过distinct对存量元素进行去重操作
3.通过map操作将字符串转成整数类型
4.借助sorted指定按照数字大小正序排列
5.使用limit截取排在前3位的元素
6.又一次使用map将id转为OfferModel对象类型
7.使用collect终止操作将最终处理后的数据收集到list中
```

输出结果：

```
[OfferModel{id=111},  OfferModel{id=193},  OfferModel{id=205}]
```

**2.4 简单结果终止流**

按照前面介绍的，终止方法里面像 count、max、min、findAny、findFirst、anyMatch、allMatch、noneMatch 等方法，均属于这里说的简单结果终止方法。所谓简单，指的是其结果形式是数字、布尔值或者 Optional 对象值等。

```
public void testSimpleStopOptions() {
    List<String> ids = Arrays.asList("205", "10", "308", "49", "627", "193", "111", "193");
    // 统计stream操作后剩余的元素个数
    System.out.println(ids.stream().filter(s -> s.length() > 2).count());
    // 判断是否有元素值等于205
    System.out.println(ids.stream().filter(s -> s.length() > 2).anyMatch("205"::equals));
    // findFirst操作
    ids.stream().filter(s -> s.length() > 2)
            .findFirst()
            .ifPresent(s -> System.out.println("findFirst:" + s));
}
```

最后结果为：

```
true
findFirst:205
```

一旦一个 Stream 被执行了终止操作之后，后续便不可以再读这个流执行其他的操作了，否则会报错，看下面示例：

```
public void testHandleStreamAfterClosed() {
    List<String> ids = Arrays.asList("205", "10", "308", "49", "627", "193", "111", "193");
    Stream<String> stream = ids.stream().filter(s -> s.length() > 2);
    // 统计stream操作后剩余的元素个数
    System.out.println(stream.count());
    System.out.println("-----下面会报错-----");
    // 判断是否有元素值等于205
    try {
        System.out.println(stream.anyMatch("205"::equals));
    } catch (Exception e) {
        e.printStackTrace();
        System.out.println(e.toString());
    }
    System.out.println("-----上面会报错-----");
}
```

结果：

```
-----下面会报错-----
java.lang.IllegalStateException: stream has already been operated upon or closed
-----上面会报错-----
java.lang.IllegalStateException: stream has already been operated upon or closed
  at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:229)
  at java.util.stream.ReferencePipeline.anyMatch(ReferencePipeline.java:516)
  at Solution_0908.main(Solution_0908.java:55)
```

因为 stream 已经被执行 count() 终止方法了，所以对 stream 再执行 anyMatch 方法的时候，就会报错 stream has already been operated upon or closed，这一点在使用的时候需要特别注意。

**2.5 结果收集终止方法**

因为 Stream 主要用于对集合数据的处理场景，所以除了上面几种获取简单结果的终止方法之外，更多的场景是获取一个集合类的结果对象，比如 List、Set 或者 HashMap 等。

这里就需要 collect 方法出场了，它可以支持生成如下类型的结果数据：

1. 一个集合类，比如 List、Set 或者 HashMap 等；

2.StringBuilder 对象，支持将多个字符串进行拼接处理并输出拼接后结果；

3. 一个可以记录个数或者计算总和的对象（数据批量运算统计）；

## 2.5.1 生成集合对象

应该算是 collect 最常被使用到的一个场景了：

```
List<NormalOfferModel> normalOfferModelList = Arrays.asList(new NormalOfferModel("11"),
                new NormalOfferModel("22"),
                new NormalOfferModel("33"));

// collect成list
List<NormalOfferModel> collectList = normalOfferModelList
        .stream()
        .filter(offer -> offer.getCate1LevelId().equals("11"))
        .collect(Collectors.toList());
System.out.println("collectList:" + collectList);

// collect成Set
Set<NormalOfferModel> collectSet = normalOfferModelList
        .stream()
        .filter(offer -> offer.getCate1LevelId().equals("22"))
        .collect(Collectors.toSet());
System.out.println("collectSet:" + collectSet);

// collect成HashMap，key为id，value为Dept对象
Map<String, NormalOfferModel> collectMap = normalOfferModelList
        .stream()
        .filter(offer -> offer.getCate1LevelId().equals("33"))
        .collect(Collectors.toMap(NormalOfferModel::getCate1LevelId, Function.identity(), (k1, k2) -> k2));
System.out.println("collectMap:" + collectMap);
```

结果：

```
collectList:[NormalOfferModel{cate1LevelId='11', imgUrl='null', linkUrl='null', ohterurl='null'}]
collectSet:[NormalOfferModel{cate1LevelId='22', imgUrl='null', linkUrl='null', ohterurl='null'}]
collectMap:{33=NormalOfferModel{cate1LevelId='33', imgUrl='null', linkUrl='null', ohterurl='null'}}
```

## 2.5.2 生成拼接字符串

将一个 List 或者数组中的值拼接到一个字符串里并以逗号分隔开，这个场景相信大家都不陌生吧？如果通过 for 循环和 StringBuilder 去循环拼接，还得考虑下最后一个逗号如何处理的问题，很繁琐:

```
public void testForJoinStrings() {
    List<String> ids = Arrays.asList("205", "10", "308", "49", "627", "193", "111", "193");
    StringBuilder builder = new StringBuilder();
    for (String id : ids) {
        builder.append(id).append(',');
    }
    // 去掉末尾多拼接的逗号
    builder.deleteCharAt(builder.length() - 1);
    System.out.println("拼接后：" + builder.toString());
}
```

但是现在有了 Stream，使用 collect 可以轻而易举的实现：

```
public void testCollectJoinStrings() {
    List<String> ids = Arrays.asList("205", "10", "308", "49", "627", "193", "111", "193");
    String joinResult = ids.stream().collect(Collectors.joining(","));
    System.out.println("拼接后：" + joinResult);
}
```

两种方式都可以得到完全相同的结果，但 Stream 的方式更优雅：

```
拼接后：205,10,308,49,627,193,111,193
```

2.5.3 批量数学运算

还有一种场景，实际使用的时候可能会比较少，就是使用 collect 生成数字数据的总和信息，也可以了解下实现方式：

```
public void testNumberCalculate() {
    List<Integer> ids = Arrays.asList(10, 20, 30, 40, 50);
    // 计算平均值
    Double average = ids.stream().collect(Collectors.averagingInt(value -> value));
    System.out.println("平均值：" + average);
    // 数据统计信息
    IntSummaryStatistics summary = ids.stream().collect(Collectors.summarizingInt(value -> value));
    System.out.println("数据统计信息：" + summary);
}
```

上面的例子中，使用 collect 方法来对 list 中元素值进行数学运算，结果如下：

```
平均值：30.0
总和：IntSummaryStatistics{count=5, sum=150, min=10, average=30.000000, max=50}
```

三、并行 Stream

**3.1parallelStream 的机制说明**

使用并行流，可以有效利用计算机的多 CPU 硬件，提升逻辑的执行速度。并行流通过将一整个 stream 划分为多个片段，然后对各个分片流并行执行处理逻辑，最后将各个分片流的执行结果汇总为一个整体流。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuRE43g97EQW40MhFdh3n4MGoeQH2a3U2bpZicmBYicZRL5g7zRpLqtefw/640?wx_fmt=other&from=appmsg)

可以通过 parallelStream 的源码发现 parallel Stream 底层是将任务进行了切分，最终将任务传递给了 jdk8 自带的 “全局”ForkJoinPool 线程池。  在 Fork-Join 中，比如一个拥有 4 个线程的 ForkJoinPool 线程池，有一个任务队列，一个大的任务切分出的子任务会提交到线程池的任务队列中，4 个线程从任务队列中获取任务执行，哪个线程执行的任务快，哪个线程执行的任务就多，只有队列中没有任务线程才是空闲的，这就是工作窃取。

可以通过下图更好的理解这种 “分而治之” 的思想：

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuribDy0at5JnFCic7aSrIMfWdnGQReUJObBNv78pO0jz0OfibTl0vxwztg/640?wx_fmt=other&from=appmsg)

**3.2 约束与限制**

1.parallelStream() 中 foreach() 操作必须保证是线程安全的；

很多人在用惯了流式处理之后，很多 for 循环都会直接使用流式 foreach(), 实际上这样不一定是合理的，如果只是简单的 for 循环，确实没有必要使用流式处理，因为流式底层封装了很多流式处理的复杂逻辑，从性能上来讲不占优。

2.parallelStream() 中 foreach() 不要直接使用默认的线程池；

```
ForkJoinPool customerPool = new ForkJoinPool(n);
customerPool.submit(
     () -> customerList.parallelStream().具体操作
```

3.parallelStream() 使用的时候尽量避免耗时操作；

如果遇到耗时的操作，或者大量 IO 的操作，或者有线程 sleep 的操作一定要避免使用并行流。

四、Stream 流可能会遇到的一些坑点

**4.1 parallelStream 和整个 java 进程共用 ForkJoinPool**

如果直接使用 parallelStream().foreach 会默认使用全局的 ForkJoinPool，而这样就会导致当前程序很多地方共用同一个线程池，包括 gc 相关操作在内，所以一旦任务队列中满了之后，就会出现阻塞的情况，导致整个程序的只要当前使用 ForkJoinPool 的地方都会出现问题。

**4.2 parallelStream 使用后 ThreadLocal 数据为空**

parallelStream 创建的并行流在真正执行时是由 ForkJoin 框架创建多个线程并行执行，由于 ThreadLocal 本身不具有可继承性，新生成的线程自然无法获取父线程中的 ThreadLocal 数据。

**4.3 转 map 的时候，没有注意到 key 值重复**

在使用 Stream 流转 map 操作的时候，需要注意 key 重复的问题。 

五、Stream 流总结

**5.1 优势**

1. 代码更简洁。偏声明式的编码风格，更容易体现出代码的逻辑意图。
2. 逻辑间解耦。一个 stream 中间处理逻辑，无需关注上游与下游的内容，只需要按约定实现自身逻辑即可。
3. 并行流场景效率会比迭代器逐个循环更高。
4. 函数式接口，延迟执行的特性，中间管道操作不管有多少步骤都不会立即执行，只有遇到终止操作的时候才会开始执行，可以避免一些中间不必要的操作消耗。

**5.2 劣势**

debug 不方便。

**创意加速器：AI 绘画创作** 

本方案展示了如何利用自研的通义万相 AIGC 技术在 Web 服务中实现先进的图像生成。其中包括文本到图像、涂鸦转换、人像风格重塑以及人物写真创建等功能。这些能力可以加快艺术家和设计师的创作流程，提高创意效率。

点击阅读原文查看详情。