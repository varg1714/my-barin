---
source: https://discord.com/blog/how-discord-stores-billions-of-messages
create: 2024-02-18 15:45
read: false
---
Discord continues to grow faster than we expected and so does our user-generated content. With more users comes more chat messages. In July, [we announced 40 million messages a day](https://blog.discordapp.com/11-million-players-in-one-year/), in December [we announced 100 million](http://venturebeat.com/2016/12/08/discord-hits-25-million-users-and-releases-gamebridge-sdk-for-its-voice-chat/), and as of this blog post we are well past 120 million. We decided early on to store all chat history forever so users can come back at any time and have their data available on any device. This is a lot of data that is ever increasing in velocity, size, and must remain available. _How do we do it? Cassandra!_  
Discord 的增长速度持续快于我们的预期，我们的用户生成内容也是如此。用户越多，聊天消息就越多。 7 月份，我们宣布每天有 4000 万条消息，12 月份我们宣布每天有 1 亿条消息，截至这篇博文，我们已经远远超过了 1.2 亿条。我们很早就决定永久存储所有聊天历史记录，以便用户可以随时返回并在任何设备上获取他们的数据。这些数据的速度和大小都在不断增加，并且必须保持可用。我们该怎么做呢？卡桑德拉！

## What we were doing 我们在做什么

The original version of Discord was built in just under two months in early 2015. Arguably, one of the best databases for iterating quickly is MongoDB. Everything on Discord was stored in a single MongoDB replica set and this was intentional, but we also planned everything for easy migration to a new database (we knew we were not going to use MongoDB sharding because it is complicated to use and not known for stability). This is actually part of our company culture: build quickly to prove out a product feature, but always with a path to a more robust solution.  
Discord 的原始版本于 2015 年初在不到两个月的时间内构建完成。可以说，快速迭代的最佳数据库之一是 MongoDB。 Discord 上的所有内容都存储在单个 MongoDB 副本集中，这是有意为之，但我们也计划了一切以便轻松迁移到新数据库（我们知道我们不会使用 MongoDB 分片，因为它使用起来很复杂，而且稳定性也不高） ）。这实际上是我们公司文化的一部分：快速构建以证明产品功能，但始终提供更强大的解决方案。

The messages were stored in a MongoDB collection with a single compound index on channel_id and created_at. Around November 2015, we reached 100 million stored messages and at this time we started to see the expected issues appearing: the data and the index could no longer fit in RAM and latencies started to become unpredictable. It was time to migrate to a database more suited to the task.  
消息存储在 MongoDB 集合中，在channel_id 和created_at 上有单个复合索引。 2015 年 11 月左右，我们存储的消息达到 1 亿条，此时我们开始看到预期的问题出现：数据和索引不再适合 RAM，延迟开始变得不可预测。是时候迁移到更适合该任务的数据库了。

## Choosing the Right Database  
选择正确的数据库

Before choosing a new database, we had to understand our read/write patterns and why we were having problems with our current solution.  
在选择新数据库之前，我们必须了解我们的读/写模式以及为什么我们当前的解决方案会出现问题。

*   It quickly became clear that our reads were extremely random and our read/write ratio was about 50/50.  
    很快我们就发现我们的读取是极其随机的，并且读/写比率约为 50/50。
*   Voice chat heavy Discord servers send almost no messages. This means they send a message or two every few days. In a year, this kind of server is unlikely to reach 1,000 messages. The problem is that even though this is a small amount of messages it makes it harder to serve this data to the users. Just returning 50 messages to a user can result in many random seeks on disk causing disk cache evictions.  
    语音聊天繁重的 Discord 服务器几乎不发送任何消息。这意味着他们每隔几天发送一两条消息。一年内，这种服务器不太可能达到 1000 条消息。问题在于，尽管消息量很小，但向用户提供这些数据却变得更加困难。仅向用户返回 50 条消息可能会导致在磁盘上进行多次随机查找，从而导致磁盘缓存被逐出。
*   Private text chat heavy Discord servers send a decent number of messages, easily reaching between 100 thousand to 1 million messages a year. The data they are requesting is usually very recent only. The problem is since these servers usually have under 100 members the rate at which this data is requested is low and unlikely to be in disk cache.  
    私人文本聊天重型 Discord 服务器发送大量消息，每年轻松达到 10 万到 100 万条消息。他们请求的数据通常只是最近的。问题是，由于这些服务器的成员通常少于 100 个，因此请求此数据的速率很低，并且不太可能位于磁盘缓存中。
*   Large public Discord servers send a lot of messages. They have thousands of members sending thousands of messages a day and easily rack up millions of messages a year. They almost always are requesting messages sent in the last hour and they are requesting them often. Because of that the data is usually in the disk cache.  
    大型公共 Discord 服务器会发送大量消息。他们有数千名会员每天发送数千条消息，并且每年轻松积累数百万条消息。他们几乎总是请求在过去一小时内发送的消息，并且经常请求这些消息。因此，数据通常位于磁盘缓存中。
*   We knew that in the coming year we would add even more ways for users to issue random reads: the ability to view your mentions for the last 30 days then jump to that point in history, viewing plus jumping to pinned messages, and full-text search. _All of this spells more random reads!!_  
    我们知道，在来年，我们将为用户添加更多随机阅读的方式：查看过去 30 天的提及，然后跳转到历史记录中的该点、查看并跳转到固定消息以及全文搜索。所有这些都意味着更多的随机阅读！

Next we defined our requirements:  
接下来我们定义了我们的要求：

*   **Linear scalability —** We do not want to reconsider the solution later or manually re-shard the data.  
    线性可扩展性 ——我们不想稍后重新考虑解决方案或手动重新分片数据。
*   **Automatic failover —** We love sleeping at night and build Discord to self heal as much as possible.  
    自动故障转移 ——我们喜欢在晚上睡觉，并构建 Discord 来尽可能地进行自我修复。
*   **Low maintenance —** It should just work once we set it up. We should only have to add more nodes as data grows.  
    维护成本低——一旦我们设置好它就应该可以工作。我们只需要随着数据的增长添加更多节点。
*   **Proven to work —** We love trying out new technology, but not too new.  
    经证实有效 ——我们喜欢尝试新技术，但又不太新。
*   **Predictable performance** **—** We have alerts go off when our API’s response time 95th percentile goes above 80ms. We also do not want to have to cache messages in Redis or Memcached.  
    可预测的性能——当 API 的响应时间第 95 个百分点超过 80 毫秒时，我们就会发出警报。我们也不希望必须在 Redis 或 Memcached 中缓存消息。
*   **Not a blob store —** Writing thousands of messages per second would not work great if we had to constantly deserialize blobs and append to them.  
    不是 blob 存储——如果我们必须不断地反序列化 blob 并附加到它们上，那么每秒写入数千条消息的效果就不会很好。
*   **Open source —** We believe in controlling our own destiny and don’t want to depend on a third party company.  
    开源 ——我们相信掌控自己的命运，不想依赖第三方公司。

Cassandra was the only database that fulfilled all of our requirements. We can just add nodes to scale it and it can tolerate a loss of nodes without any impact on the application. Large companies such as Netflix and Apple have thousands of Cassandra nodes. Related data is stored contiguously on disk providing minimum seeks and easy distribution around the cluster. It’s backed by DataStax, but still open source and community driven.  
Cassandra 是唯一满足我们所有要求的数据库。我们只需添加节点即可对其进行扩展，并且它可以容忍节点的丢失，而不会对应用程序产生任何影响。 Netflix 和 Apple 等大公司拥有数千个 Cassandra 节点。相关数据连续存储在磁盘上，提供最少的查找次数并轻松分布在集群周围。它由 DataStax 支持，但仍然是开源和社区驱动的。

Having made the choice, we needed to prove that it would actually work.  
做出选择后，我们需要证明它确实有效。

## Data Modeling 数据建模

The best way to describe Cassandra to a newcomer is that it is a KKV store. The two Ks comprise the primary key. The first K is the partition key and is used to determine which node the data lives on and where it is found on disk. The partition contains multiple rows within it and a row within a partition is identified by the second K, which is the clustering key. The clustering key acts as both a primary key within the partition and how the rows are sorted. You can think of a partition as an ordered dictionary. These properties combined allow for very powerful data modeling.  
对新手描述 Cassandra 的最好方式就是它是一家 KKV 商店。两个 K 构成主键。第一个 K 是分区键，用于确定数据位于哪个节点以及在磁盘上的位置。分区中包含多行，分区中的一行由第二个 K（集群键）标识。聚集键既充当分区内的主键，又充当行的排序方式。您可以将分区视为有序字典。这些属性相结合可以实现非常强大的数据建模。

Remember that messages were indexed in MongoDB using channel_id and created_at? channel_id became the partition key since all queries operate on a channel, but created_at didn’t make a great clustering key because two messages can have the same creation time. Luckily every ID on Discord is actually a [Snowflake](https://blog.twitter.com/2010/announcing-snowflake) (chronologically sortable), so we were able to use them instead. The primary key became (channel_id, message_id), where the message_id is a Snowflake. This meant that when loading a channel we could tell Cassandra exactly where to range scan for messages.  
还记得 MongoDB 中的消息是使用channel_id 和created_at 建立索引的吗？由于所有查询都在通道上运行，channel_id 成为分区键，但created_at 并没有成为一个很好的集群键，因为两条消息可以具有相同的创建时间。幸运的是，Discord 上的每个 ID 实际上都是雪花（按时间顺序排序），因此我们可以使用它们。主键变成了(channel_id, message_id)，其中message_id是一个Snowflake。这意味着在加载通道时我们可以告诉 Cassandra 在哪里进行范围扫描以查找消息。

Here is a simplified schema for our messages table (this omits about 10 columns).  
这是我们的消息表的简化架构（省略了大约 10 列）。

<table data-hpc="" data-tab-size="8" data-paste-markdown-skip="" data-tagsearch-lang="SQL" data-tagsearch-path="schema1.cql"><tbody><tr><td data-line-number="1"></td><td><span>CREATE</span> <span>TABLE</span> <span>messages</span> (</td></tr><tr><td data-line-number="2"></td><td>&nbsp; channel_id <span>bigint</span>,</td></tr><tr><td data-line-number="3"></td><td>message_id <span>bigint</span>,</td></tr><tr><td data-line-number="4"></td><td>author_id <span>bigint</span>,</td></tr><tr><td data-line-number="5"></td><td>content <span>text</span>,</td></tr><tr><td data-line-number="6"></td><td><span>PRIMARY KEY</span> (channel_id, message_id)</td></tr><tr><td data-line-number="7"></td><td>) WITH CLUSTERING <span>ORDER BY</span> (message_id <span>DESC</span>);</td></tr></tbody></table>

While Cassandra has schemas not unlike a relational database, they are cheap to alter and do not impose any temporary performance impact. We get the best of a blob store and a relational store.  
虽然 Cassandra 的模式与关系数据库没有什么不同，但它们的更改成本很低，并且不会造成任何临时的性能影响。我们充分利用了 Blob 存储和关系存储。

When we started importing existing messages into Cassandra we immediately began to see warnings in the logs telling us that partitions were found over 100MB in size. _What gives?!_ _Cassandra advertises that it can support 2GB partitions!_ Apparently, just because it can be done, it doesn’t mean it should. Large partitions put a lot of GC pressure on Cassandra during compaction, cluster expansion, and more. Having a large partition also means the data in it cannot be distributed around the cluster. It became clear we had to somehow bound the size of partitions because a single Discord channel can exist for years and perpetually grow in size.  
当我们开始将现有消息导入 Cassandra 时，我们立即开始在日志中看到警告，告诉我们发现分区大小超过 100MB。是什么赋予了？！ Cassandra宣传可以支持2GB分区！显然，仅仅因为可以做到，并不意味着应该这样做。大型分区在压缩、集群扩展等过程中给 Cassandra 带来了很大的 GC 压力。拥有大分区也意味着其中的数据无法分布在集群中。很明显，我们必须以某种方式限制分区的大小，因为单个 Discord 通道可以存在多年，并且大小会不断增长。

We decided to bucket our messages by time. We looked at the largest channels on Discord and determined if we stored about 10 days of messages within a bucket that we could comfortably stay under 100MB. Buckets had to be derivable from the message_id or a timestamp.  
我们决定按时间存储我们的消息。我们查看了 Discord 上最大的频道，并确定我们是否可以在一个存储桶中存储大约 10 天的消息，并且可以轻松地保持在 100MB 以下。存储桶必须可以从 message_id 或时间戳中派生出来。

<table data-hpc="" data-tab-size="8" data-paste-markdown-skip="" data-tagsearch-lang="Python" data-tagsearch-path="buckets.py"><tbody><tr><td data-line-number="1"></td><td><span>DISCORD_EPOCH</span> <span>=</span> <span>1420070400000</span></td></tr><tr><td data-line-number="2"></td><td><span>BUCKET_SIZE</span> <span>=</span> <span>1000</span> <span>*</span> <span>60</span> <span>*</span> <span>60</span> <span>*</span> <span>24</span> <span>*</span> <span>10</span></td></tr><tr><td data-line-number="3"></td><td></td></tr><tr><td data-line-number="4"></td><td></td></tr><tr><td data-line-number="5"></td><td><span>def</span> <span>make_bucket</span>(<span>snowflake</span>):</td></tr><tr><td data-line-number="6"></td><td><span>if</span> <span>snowflake</span> <span>is</span> <span>None</span>:</td></tr><tr><td data-line-number="7"></td><td><span>timestamp</span> <span>=</span> <span>int</span>(<span>time</span>.<span>time</span>() <span>*</span> <span>1000</span>) <span>-</span> <span>DISCORD_EPOCH</span></td></tr><tr><td data-line-number="8"></td><td><span>else</span>:</td></tr><tr><td data-line-number="9"></td><td><span># When a Snowflake is created it contains the number of</span></td></tr><tr><td data-line-number="10"></td><td><span># seconds since the DISCORD_EPOCH.</span></td></tr><tr><td data-line-number="11"></td><td><span>timestamp</span> <span>=</span> <span>snowflake_id</span> <span>&gt;&gt;</span> <span>22</span></td></tr><tr><td data-line-number="12"></td><td><span>return</span> <span>int</span>(<span>timestamp</span> <span>/</span> <span>BUCKET_SIZE</span>)</td></tr><tr><td data-line-number="13"></td><td></td></tr><tr><td data-line-number="14"></td><td></td></tr><tr><td data-line-number="15"></td><td><span>def</span> <span>make_buckets</span>(<span>start_id</span>, <span>end_id</span><span>=</span><span>None</span>):</td></tr><tr><td data-line-number="16"></td><td><span>return</span> <span>range</span>(<span>make_bucket</span>(<span>start_id</span>), <span>make_bucket</span>(<span>end_id</span>) <span>+</span> <span>1</span>)</td></tr></tbody></table>

Cassandra partition keys can be compounded, so our new primary key became ((channel_id, bucket), message_id).  
Cassandra 分区键可以复合，因此我们的新主键变为 ((channel_id, Bucket), message_id)。

<table data-hpc="" data-tab-size="8" data-paste-markdown-skip="" data-tagsearch-lang="SQL" data-tagsearch-path="schema2.cql"><tbody><tr><td data-line-number="1"></td><td><span>CREATE</span> <span>TABLE</span> <span>messages</span> (</td></tr><tr><td data-line-number="2"></td><td>channel_id <span>bigint</span>,</td></tr><tr><td data-line-number="3"></td><td>bucket <span>int</span>,</td></tr><tr><td data-line-number="4"></td><td>message_id <span>bigint</span>,</td></tr><tr><td data-line-number="5"></td><td>author_id <span>bigint</span>,</td></tr><tr><td data-line-number="6"></td><td>content <span>text</span>,</td></tr><tr><td data-line-number="7"></td><td><span>PRIMARY KEY</span> ((channel_id, bucket), message_id)</td></tr><tr><td data-line-number="8"></td><td>) WITH CLUSTERING <span>ORDER BY</span> (message_id <span>DESC</span>);</td></tr></tbody></table>

To query for recent messages in the channel we generate a bucket range from current time to channel_id (it is also a Snowflake and has to be older than the first message). We then sequentially query partitions until enough messages are collected. The downside of this method is that rarely active Discords will have to query multiple buckets to collect enough messages over time. In practice this has proved to be fine because for active Discords enough messages are usually found in the first partition and they are the majority.  
为了查询通道中的最新消息，我们生成一个从当前时间到channel_id的存储桶范围（它也是一个雪花，并且必须比第一条消息更旧）。然后，我们顺序查询分区，直到收集到足够的消息。这种方法的缺点是，很少活跃的 Discords 必须查询多个存储桶才能随着时间的推移收集足够的消息。在实践中，这已被证明是很好的，因为对于活跃的 Discord，通常可以在第一个分区中找到足够的消息，并且它们占大多数。

Importing messages into Cassandra went without a hitch and we were ready to try in production.  
将消息导入 Cassandra 顺利进行，我们已准备好在生产中进行尝试。

## Dark Launch 黑暗发射

Introducing a new system into production is always scary so it’s a good idea to try to test it without impacting users. We setup our code to double read/write to MongoDB and Cassandra.  
将新系统引入生产总是令人恐惧的，因此最好在不影响用户的情况下对其进行测试。我们将代码设置为对 MongoDB 和 Cassandra 进行双重读/写。

Immediately after launching we started getting errors in our bug tracker telling us that author_id was null. _How can it be null? It is a required field!_  
启动后，我们的错误跟踪器立即开始收到错误，告诉我们author_id为空。怎么可能为空呢？这是必填字段！

## Eventual Consistency 最终一致性

Cassandra is an [AP](https://en.wikipedia.org/wiki/CAP_theorem) database which means it trades strong consistency for availability which is something we wanted. It is an anti-pattern to read-before-write (reads are more expensive) in Cassandra and therefore everything that Cassandra does is essentially an upsert even if you provide only certain columns. You can also write to any node and it will resolve conflicts automatically using “last write wins” semantics on a per column basis. _So how did this bite us?_  
Cassandra 是一个 AP 数据库，这意味着它以强一致性换取可用性，这正是我们想要的。这是 Cassandra 中先读后写的反模式（读取成本更高），因此即使您只提供某些列，Cassandra 所做的一切本质上都是更新插入。您还可以写入任何节点，它将在每列的基础上使用“最后写入获胜”语义自动解决冲突。那么它是如何咬我们的呢？

![](https://assets-global.website-files.com/5f9072399b2640f14d6a2bf4/612414b8fa047b3fd3ce0b2a_1*t7VkLRKZVeHb_c6Tl1heew.gif)

Example of edit/delete race condition  
编辑/删除竞争条件的示例

In the scenario that a user edits a message at the same time as another user deletes the same message, we ended up with a row that was missing all the data except the primary key and the text since all Cassandra writes are upserts. There were two possible solutions for handling this problem:  
在一个用户编辑消息的同时另一个用户删除同一条消息的情况下，我们最终会得到一行缺少除主键和文本之外的所有数据的情况，因为所有 Cassandra 写入都是更新插入。处理这个问题有两种可能的解决方案：

1.  Write the whole message back when editing the message. This had the possibility of resurrecting messages that were deleted and adding more chances for conflict for concurrent writes to other columns.  
    编辑消息时写回整个消息。这有可能复活已删除的消息，并增加对其他列的并发写入发生冲突的更多机会。
2.  Figuring out that the message is corrupt and deleting it from database.  
    确定消息已损坏并将其从数据库中删除。

We went with the second option, which we did by choosing a column that was required (in this case author_id) and deleting the message if it was null.  
我们选择了第二个选项，通过选择所需的列（在本例中为author_id）并删除消息（如果该列为空）来实现。

While solving this problem, we noticed we were being very inefficient with our writes. Since Cassandra is eventually consistent it cannot just delete data immediately. It has to replicate deletes to other nodes and do it even if other nodes are temporarily unavailable. Cassandra does this by treating deletes as a form of write called a “tombstone.” On read, it just skips over tombstones it comes across. Tombstones live for a configurable amount of time (10 days by default) and are permanently deleted during compaction when that time expires.  
在解决这个问题时，我们注意到我们的写入效率非常低。由于 Cassandra 最终是一致的，它不能立即删除数据。即使其他节点暂时不可用，它也必须将删除复制到其他节点并执行此操作。 Cassandra 通过将删除视为一种称为“墓碑”的写入形式来实现此目的。在阅读时，它只是跳过它遇到的墓碑。墓碑的生存时间可配置（默认为 10 天），并在该时间到期时在压缩过程中永久删除。

Deleting a column and writing null to a column are the exact same thing. They both generate a tombstone. Since all writes in Cassandra are upserts, that means you are generating a tombstone even when writing null for the first time. In practice, our entire message schema contains 16 columns, but the average message only has 4 values set. We were writing 12 tombstones into Cassandra most of the time for no reason. The solution to this was simple: only write non-null values to Cassandra.  
删除列和向列写入 null 是完全相同的事情。他们都会生成一个墓碑。由于 Cassandra 中的所有写入都是更新插入，这意味着即使第一次写入 null 也会生成墓碑。实际上，我们的整个消息模式包含 16 列，但平均消息只有 4 个值集。大多数时候我们无缘无故地将 12 个墓碑写入 Cassandra。解决方案很简单：只将非空值写入 Cassandra。

## Performance 表现

Cassandra is known to have faster writes than reads and we observed exactly that. Writes were sub-millisecond and reads were under 5 milliseconds. We observed this regardless of what data was being accessed, and performance stayed consistent during a week of testing. _Nothing was surprising, we got exactly what we expected._  
众所周知，Cassandra 的写入速度比读取速度快，我们也确实观察到了这一点。写入时间低于毫秒，读取时间低于 5 毫秒。无论访问什么数据，我们都观察到这一点，并且在一周的测试期间性能保持一致。没有什么令人惊讶的，我们得到了我们所期望的。

![](https://assets-global.website-files.com/5f9072399b2640f14d6a2bf4/612414b89ad013844eddfbd2_1*MrdDaSA6ghOQQ7WyzqztcQ.png)

Read/Write Latency via Datadog  
通过 Datadog 的读/写延迟

In line with fast, consistent read performance, here’s an example of a jump to a message from over a year ago in a channel with millions of messages:  
为了实现快速、一致的读取性能，以下是在包含数百万条消息的通道中跳转到一年多前的消息的示例：

![](https://assets-global.website-files.com/5f9072399b2640f14d6a2bf4/612414b8a31dd422e62ac9dc_1*TGzjmCHRqrWSgarkjUrAwQ.gif)

_Jumping back one full year of chat  
回溯一整年的聊天记录_

## The Big Surprise 大惊喜

Everything went smoothly, so we rolled it out as our primary database and phased out MongoDB within a week . It continued to work flawlessly…for about 6 months until that one day where Cassandra became unresponsive.  
一切都很顺利，因此我们将其作为我们的主要数据库推出，并在一周内逐步淘汰了 MongoDB。它继续完美地工作......大约 6 个月，直到有一天 Cassandra 变得毫无反应。

We noticed Cassandra was running 10 second _“stop-the-world”_ GC constantly but we had no idea why. We started digging and found a Discord channel that was taking 20 seconds to load. The [Puzzles & Dragons Subreddit](https://www.reddit.com/r/PuzzleAndDragons/) public Discord server was the culprit. Since it was public we joined it to take a look. To our surprise, the channel had only 1 message in it. It was at that moment that it became obvious they deleted millions of messages using our API, leaving only 1 message in the channel.  
我们注意到 Cassandra 不断运行 10 秒的“stop-the-world”GC，但我们不知道为什么。我们开始挖掘，发现一个 Discord 频道需要 20 秒才能加载。罪魁祸首是《智龙迷城》Subr​​eddit 公共 Discord 服务器。既然它是公开的，我们就加入进来看看。令我们惊讶的是，该频道中只有 1 条消息。就在那一刻，很明显他们使用我们的 API 删除了数百万条消息，频道中只留下了 1 条消息。

If you have been paying attention you might remember how Cassandra handles deletes using tombstones (mentioned in **Eventual Consistency**). When a user loaded this channel, even though there was only 1 message, Cassandra had to effectively scan millions of message tombstones (generating garbage faster than the JVM could collect it).  
如果您一直在关注，您可能还记得 Cassandra 如何使用逻辑删除来处理删除（在最终一致性中提到）。当用户加载此通道时，即使只有 1 条消息，Cassandra 也必须有效扫描数百万条消息墓碑（生成垃圾的速度比 JVM 收集垃圾的速度快）。

We solved this by doing the following:  
我们通过执行以下操作解决了这个问题：

*   We lowered the lifespan of tombstones from 10 days down to 2 days because we run [Cassandra repairs](https://docs.datastax.com/en/cassandra/2.1/cassandra/tools/toolsRepair.html#toolsRepair__description) (an anti-entropy process) every night on our message cluster.  
    我们将墓碑的寿命从 10 天降低到 2 天，因为我们每晚都会在消息集群上运行 Cassandra 修复（反熵过程）。
*   We changed our query code to track empty buckets and avoid them in the future for a channel. This meant that if a user caused this query again then at worst Cassandra would be scanning only in the most recent bucket.  
    我们更改了查询代码以跟踪空存储桶并在将来的通道中避免使用它们。这意味着，如果用户再次引发此查询，那么在最坏的情况下，Cassandra 将仅在最近的存储桶中进行扫描。

## The Future 未来

We are currently running a 12 node cluster with a replica factor of 3 and will just continue to add new Cassandra nodes as needed. We believe this will continue to work for a long time but as Discord continues to grow there is a distant future where we are storing billions of messages per day. Netflix and Apple run clusters of hundreds of nodes so we know we can punt thinking too much about this for a while. However we like to have some ideas in our pocket for the future.  
我们目前正在运行一个副本因子为 3 的 12 节点集群，并将根据需要继续添加新的 Cassandra 节点。我们相信这将持续很长一段时间，但随着 Discord 的不断增长，在遥远的未来我们每天会存储数十亿条消息。 Netflix 和 Apple 运行着由数百个节点组成的集群，因此我们知道我们可以在一段时间内对此进行过多思考。然而，我们希望对未来有一些想法。

### Near term 短期

*   Upgrade our message cluster from Cassandra 2 to Cassandra 3. Cassandra 3 has a [new storage format](http://www.datastax.com/2015/12/storage-engine-30) that can reduce storage size by more than 50%.  
    将我们的消息集群从 Cassandra 2 升级到 Cassandra 3。Cassandra 3 具有新的存储格式，可以将存储大小减少 50% 以上。
*   Newer versions of Cassandra are better at handling more data on a single node. We currently store nearly 1TB of compressed data on each node. We believe we can safely reduce the number of nodes in the cluster by bumping this to 2TB.  
    新版本的 Cassandra 更擅长在单个节点上处理更多数据。目前我们在每个节点上存储了近 1TB 的压缩数据。我们相信，通过将其增加到 2TB，我们可以安全地减少集群中的节点数量。

### Long term 长期

*   Explore using [Scylla](http://www.scylladb.com/), a Cassandra compatible database written in C++. During normal operations our Cassandra nodes are actually not using too much CPU, however at non peak hours when we run repairs (an anti-entropy process) they become fairly CPU bound and the duration increases with the amount of data written since the last repair. Scylla advertises significantly lower repair times.  
    使用 Scylla 进行探索，这是一个用 C++ 编写的兼容 Cassandra 的数据库。在正常操作期间，我们的 Cassandra 节点实际上并没有使用太多的 CPU，但是在非高峰时段，当我们运行修复（反熵进程）时，它们会相当受 CPU 限制，并且持续时间随着自上次修复以来写入的数据量而增加。 Scylla 宣称修复时间显着缩短。
*   Build a system to archive unused channels to flat files on Google Cloud Storage and load them back on-demand. We want to avoid doing this one and don’t think we will have to do it.  
    构建一个系统，将未使用的频道存档到 Google Cloud Storage 上的平面文件中，并按需加载它们。我们想避免这样做，也不认为我们必须这样做。

## 结语

It has now been just over a year since we made the switch and, despite _“the big surprise,”_ it has been smooth sailing. We went from over 100 million total messages to more than 120 million messages a day, with performance and stability staying consistent.  
自从我们做出转变以来已经过去了一年多的时间，尽管有“巨大的惊喜”，但一切进展顺利。我们每天的消息总数从超过 1 亿条增加到超过 1.2 亿条，并且性能和稳定性保持一致。

Due to the success of this project we have since moved the rest of our live production data to Cassandra and that has also been a success.  
由于该项目的成功，我们已将其余实时生产数据转移到 Cassandra，这也取得了成功。

In a follow-up to this post we will explore how we make billions of messages searchable.  
在这篇文章的后续内容中，我们将探讨如何使数十亿条消息变得可搜索。

We don’t have dedicated DevOps engineers yet (only 4 backend engineers), so having a system we don’t have to worry about has been great. _We are hiring, so_ [_come join us_](https://discordapp.com/jobs) _if this type of stuff tickles your fancy._  
我们还没有专门的 DevOps 工程师（只有 4 名后端工程师），因此拥有一个我们不必担心的系统真是太棒了。我们正在招聘，所以如果您喜欢这种类型的东西，请加入我们。