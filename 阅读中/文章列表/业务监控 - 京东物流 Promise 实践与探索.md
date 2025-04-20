---
source: https://mp.weixin.qq.com/s/u2JVdK9BaQSKy5Byxe8Ssg
create: 2025-04-17 21:40
read: false
tags:
  - 系统架构
---
**一、业务监控的意义  
**

指标是一个被定义的数值，用来对事实进行量化和抽象**。**作为技术人员来说，我们需要共同考虑技术指标和业务指标。

## **1）技术指标**

技术指标定义了服务可用率、性能 TP99、调用量等技术指标。这些指标能够帮助开发人员深入了解系统的运行状态，及时发现并解决潜在问题。

虽然技术指标正常是系统稳定性的一个重要参考，但并不能完全保证业务无异常。业务异常可能源于多种非技术因素，如业务流程错误、用户需求变化、外部依赖问题、人为因素（配置错误）等。因此，在监控技术指标的同时，还需要结合业务层面的监控和分析手段，以确保全面了解和掌握系统的运行状态，从而保障业务的稳定性和可靠性。  

## **2）业务指标**

业务指标关注的是数据的正确性和完整性。业务指标的重要性不容忽视，它在系统稳定性管理和数据驱动的决策制定中扮演着关键的角色。

**1）提前发现线上问题**

通过业务指标体系，我们可以揭示系统技术问题或业务数据中的问题，提前发现并解决问题，缩短线上平均修复时长（MTTR）。

**2）掌握业务运营规律**

通过对业务指标的监控和分析，我们可以更好地理解业务运营的规律和趋势。例如，通过监控订单量、物流配送时间等指标，可以更好地规划时效和调整策略。

**3）驱动业务运营**

业务监控不仅是被动的监控和反应，还可以积极地驱动业务运营。例如，如果发现某个地区的物流配送时间长于预期，可以与物流供应商合作，优化配送路线或增加配送资源，以提高客户满意度并降低投诉率。  

## **3）技术指标 & 业务指标关系**

技术指标和业务监控的数据正确性通常是相互关联的。

技术指标不可用，则肯定业务指标会不可用。相反如果业务指标不可用有异常，不一定技术指标不可用。例如，如果一个系统的可用性低于 SLA 中定义的阈值，那么这可能会影响到业务流程的正常运行，从而导致数据错误或丢失。

1 个技术指标可对应 1 个 / N 个业务指标，针对返回数据加工为业务指标可能比较简单，有些可能复杂。当然 N 个技术指标对应 1 个或者 M 个业务指标。  

# **二、业务指标体系搭建的基本流程**

## **1）确定业务指标对应的价值**

此阶段是搭建业务指标体系的基石，研发需要有业务的思维，对系统的业务逻辑理解越全面，指标搭建就越合理。需要深入思考以下几点？

1.【有价值】你的应用 & 服务接口的核心价值是什么？这个指标可以体现这种价值吗？

2.【可度量】这个指标是不是可以判断接口数据的准确性？可以提前发现系统或者业务配置问题。

3.【可操作】这个指标是不是一个可操作的指标？如果一个指标变坏，你什么也做不了，那它对你来说相当于不存在

4.【可理解】这个指标是不是很容易被你的整个团队理解呢？

指标搭建没有特别高大上的模型和理论依托，但是却有赖于深厚的积累和充分的业务理解。只要能达到反映业务真实现状，方便及时定位异常点就好。这个体系是动态的，可以随着不同阶段的业务需要不断进行更新和调整，也不断在全面和精炼中寻找平衡，避免过高的复杂度带来的冗余。  

 **2）业务指标设计**

### **2.1）业务指标分类**

业务指标可以根据其性质和用途被分类为：

1. 基础指标：这是最基本的指标，业务实体原子量化属性的且不可再分的概念集合

2. 复合指标：指建立在基础指标之上，通过一定运算规则形成的计算指标集合

3. 派生指标：指基础指标或复合指标与维度成员、统计属性等相结合产生的指标，例如累计值，同比

### **2.2）什么样的指标算一个好的指标？**

1. 明确性：好的指标需要有明确的定义和计算方法，这样才能确保其可被理解和操作。

2. 可操作性：指标应能引导行动，即它应能驱动实际的操作或决策。这意味着指标的结果应当是可执行的。

3. 可比性：通过比较不同时间段或不同群体的数据，可以更好地理解业务的表现和趋势。

4. 简单性：好的指标应该是一个简单的数字，能够快速统计。

5. 可监控性：业务指标需要有明显的规律可循，具有监控意义。

## **3）业务指标监控的方法与工具**

在业务指标建设这个过程中，我们更多的是从使用者（用户、一线业务运营）这个视角去看监控。常用的业务指标监控方法

• 同比环比法：通过和过去的自己比较，明确当前指标的水平和变化趋势。

• 标准差法：通过比较变化幅度与历史数据标准差，设计预警机制。

• 智能化的阈值预警：根据业务指标历史经验来调整不同时间段的阈值。

## **4）业务指标报警后跟进**

收到业务指标的报警后，分析告警原因：是正常逻辑、技术代码问题、上游输入参数问题或业务配置问题。  

# **三、用户 (业务) 视角关注什么**

Promise 是一个对外时效承诺，对内控制订单生产节奏。

1）从用户（业务）视角业务图如下，对外提供时效日历，需要监控日历天数的有效性， 日历波次的可用性。

2）从物流（仓库作业业务）视角来看，对内订单控制生产，需要监控订单的生产节奏（生产太快导致爆仓，生产太慢导致仓库作业人员无订单生产）

  

# **四、实践落地 - 迭代完善**

## 1）小步快跑

业务监控刚开始摸索阶段，先干为主（实践中再模式模型抽象），先覆盖 P3/P4 级事故监控，逐步迭代、改进、完善。

### 1.1）从无到有

构建一个有效的业务指标是一个循序渐进的过程。下图是一个监控数据同步成功率的业务指标，由于存在入参异常等噪音数据，导致成功率波动较大，起初并不具备监控意义。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWa82icdMbJM1mxaeVE8BLzxia9ZtXd7VlQr8XRGXTBPM9vzSwwS1CtICjKnjZqMcv5XlwAmP0FWABQ/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

 1.2）从有到准

经过对异常数据的分析归类，通过结果列列值翻译功能，将与业务无关的异常映射为成功，去除干扰因素后，该指标变得基本稳定，具有监控意义。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWa82icdMbJM1mxaeVE8BLzxF3gjsWDNTqWiaIrTia2sGZaOG6ibWletDw2GJwC2S9gTqg3c4iaVuoWxNw/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

 去除干扰数据后，该指标变得基本稳定，具有监控意义。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWa82icdMbJM1mxaeVE8BLzx5HF12jEWByO2B1icN3LEL1quwguycYyiatEj42lZPeiaBTUmuhbfveydQ/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

 2）模型完善

### 2.1）下单前 - 结算日历

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWa82icdMbJM1mxaeVE8BLzxyq95rf01mVx4x3RCibr79uC48bB6iacPs44SqnN5UvcGskuZBlkwiaqNQ/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

 日历做业务监控比较早，之前是用的 pfinder 监控的，效果图如下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWa82icdMbJM1mxaeVE8BLzxGMyziat9v8GyOIs3p9HqPMiarF4UroT39FicMibibacM6wRhvNNXVo0xmvw/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

 2.2）下单后 - 订单下传率

1）覆盖场景：业务配置数据不准（比如配提前了或者延后了）

2）效果图如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWa82icdMbJM1mxaeVE8BLzxjR5ot5f4bqNDP3ZLyticicBI8SaKTRSuRG1GuqZmkUEF8YLbX5iaqQuhw/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

 3）最终打印的业务监控日志如下：

```
|xx服务>tid=xxxx>orderId=xxxxxxxx>|transferService|-1|Tue Jan 07 00:00:00 CST 2025

```

5）监控配置如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWa82icdMbJM1mxaeVE8BLzxysm3icibJoV1OBCYJMZNGrjEEx0hiazIj1Bk03tkK6tzYrXXl1KOqlic2A/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

 2.3）售后 - 逆向取件日历

对外界面如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWa82icdMbJM1mxaeVE8BLzxCudKFX6ibht5kibjFoJG4hFsIgg6catgVELWuuHJ8XibZl9Q0lYgI2u6g/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

 针对预约日历，监控其有效性（如是否有日期可选、日历长度是否满足、是否有可选的波次等），以帮助业务、技术及时发现系统潜在风险，有异常时快速排查。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWa82icdMbJM1mxaeVE8BLzxV0QM6ORKNDXiavcUxkCATs1XwAsuAeURSyUKUVHgE87m33WrVu8MzibQ/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

 监控如下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWa82icdMbJM1mxaeVE8BLzxwsdKCSm3nnvlOfCcmv8RVGhQvLQCzuem4eqwbzyjppicARULbWjQ3Fg/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

 3）京 ME 直达一线业务

描述：当门店某个波次的订单量在其预设产能的 120% 以内时视为正常波动，而超出这个值则需要及时告知仓库 / 门店 具体运营同学进行调控。

设计：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BWa82icdMbJM1mxaeVE8BLzx1QPjrsl2AZufI3V60s1H6Lfyl675Rl5oUk9aXezHTHg87WydcF8QDQ/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

**五、常见误区**

## 1. 指标太多

业务监控的目的是发现问题，而不是指标越多越好

## 2、指标定义不清楚

事实上指标的定义没有绝对的正确和错误，只要基于实际业务问题，确定好统计口径，团队理解一致，这个指标就是可取的。所以对于一些界定不够清晰，不能被团队一眼理解的指标，最好再追加基本的解释。

案例：大家都知道看似简单的 DAU，其实也可以有不同的定义。什么是活跃？最常见的定义是，打开应用即算活跃。有些应用是登录或者停留才算活跃。那么 DAU 的定义是登陆你的 app or 停留时长达到一分钟 or 有特定行为？其实都可以，只要团队一致认可且能反映产品的核心价值。

## 3、宁少勿滥

在不影响完整性的情况下，能少则少，能用一项指标说明问题绝不用两项。

## 4、业务监控指标需要快速定位问题

别业务监控报警出来了，但无法快速定位问题，那这业务监控指标的意义就大打折扣了，故设计业务监控指标的时候，一定要思考，我这业务监控指标报警了，我如何快速定位问题，比如业务监控日志需要打印，订单、以及全链路 traceid，仓库，四级地址、错误码等关键信息。一遇到报警查看业务监控日志可快速定位

## 5、业务监控指标代码 不要影响技术可用率

千万别应用添加业务监控代码，而导致系统技术本身问题。比如业务监控抛异常等，故我是要求团队业务监控代码必须 catch 异常，不能影响现有代码逻辑

  

# **六、未来规划**

1. 优化现有业务指标：我们将继续精细化打磨现有的业务指标，确保报警系统能够更加快速、准确地响应，提升整体效率和可靠性。

2. 构建外单链路业务指标：我们计划建设和完善外单链路的业务指标体系，以全面监控和分析外部订单链路的运行情况。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9K73WSRq6BXGicp40bMAicmX9DpEDjMlfPJT23acLpRzmuyiaguHv0VlmVDyEFGwd36gZYRShzhv0EPleicHyvk7KA/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=wxpic)

  

扫一扫，加入技术交流群