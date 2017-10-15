---
title: 开放式 ETL 系统的设计与实践
date: 2017-09-20 15:42:33
tags: 软文收集
---

## 开放式 ETL 系统的设计与实践

大家晚上好，感谢大家能来参加本次的在线分享，首先自我介绍一下，我是14年校招加入百度，之前在百度质量部参与EP相关的一些项目研发，15年加入百度外卖，主要负责数据仓库和ETL以及部分数据产品的开发。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVugFVoWbNuSpE1GPdGZHvjmTqic8NopmHeRY4CgL0cAFW51deqUdUyvg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

今天主要给大家介绍一下百度外卖这边在ETL方向的一些实践。整体分四块，第一块先介绍下ETL的需求来源，以及外卖这个业务在ETL中的一些场景特征，这些场景特征和后面讲到的ETL系统设计联系比较紧密。第二部分讲基于这些场景和业务，ETL都需要满足哪些特性，都针对性的做出了什么样的设计。第三部分介绍了整个数据的交付体系，在数据交付给业务使用时如何保证其可用性，如何做到生产进度透明化。第四部分是介绍如何通过数据的血缘关系，来做到数据可解释，同时基于数据血缘关系还可以做那些事，能带来什么样的特性。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVFhMLzuHD9WCVtUSE4WoywJZQNZA7Y0VwicwonebJBDiamT8tia4jibZQFg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

聊ETL自然离不开仓库，ETL服务将数据入到仓库，最终也都是为了在仓库层面提供结构或半结构化的数据存储、提供面向主题的明细和聚合数据，更好的支持上层应用及业务。

外卖这边数据需求（也就是数据出口）主要分三大类：一，分析类需求。相信大多数业务后端RD都做过给老板或PM跑数的需求，其实这就是最原始的ETL需求，外卖这边每日的业务整体日报，分城市、分代理商的日报，以及各种通过“自如”自定义或订阅的报表，都是基于ETL系统在释放数据可用信号后开始跑相应的报表服务来实现依赖发送的。二，线上业务产品的需求。比如在面向商户的产品中，展示商户的流量、复购、潜在顾客等相关dashboard，都是基于仓库的数据统计分析出来的。再比如下发给bd的商机系统，为销售提供销售线索的指标；面向bd的kpi考核，kpi每个指标的口径定义，具体实现就是对应在ETL的处理逻辑上。除产品研发相关的业务外，同时支持公司内其他业务部门，如财务、预算等等。三，就是最能挖掘数据潜在价值的策略方向的应用，支撑rank、搜索推荐、画像等。良好的结构化数据和仓库层次划分，能极大减小数据挖掘在前期清洗和预处理的成本，同时提供便捷的数据使用方式，让业务部门能更加高效的专注于自己的业务领域。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVoJJldy1KaXb9gBYbvcQdSq8Bxicrkxty0zibicACjMMpiaE8hC6tr7tq5w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

针对以上业务场景，在ETL中又有着怎样的数据场景，有哪些特别的地方呢，我们从这三个视角来看。

首先，从数据流上来讲，数据主要有两种流向，一种是从业务库中整表抽取或部分字段抽取。整表抽取，主要是业务端的一些维度表，量级不大，变更频率低。分字段抽取，这个是核心场景，一般的业务主体数据在OLTP场景中都会细分业务进行分库分表，那么对应在仓库中的一个主题数据，比如订单，其基础指标在业务库中就可能存的比较分散，比如订单ID、价格、状态等在一个表，订单退款详情、评论、投诉等信息在另一个库另几张表，那么就需要完成一个跨库跨表按订单id抽取后数据拼装的过程。另一种数据流向就是仓库内部表之间的流转，比如汇总表的指标需要用事实表和维度表进行关联后聚合得出，再比如商户事实表的商户日流水指标，需要订单表和商户维度表关联后聚合产出。

从逻辑视角来看，有两种逻辑映射场景，比如价格字段，在仓库的指标中细分为优惠前单价、优惠后单价，优惠前单价直接取业务库订单表中的“订单价格”即可，这是个1对1的映射关系；优惠后单价，就需要“订单价格”减去“商户补贴”减去“平台补贴”减去“支付补贴”，这是个多对一的关系。这里就涉及到一个细节问题，加入上述三个补贴字段在业务库的订单表中都是以单个字段形式存在于schema中，多对一的计算直接一个select sql就能计算出来，但如果是补贴字段放在一个压缩的json字符串中呢，就需要先解析后逐一取出再计算，就涉及到需要编码来解决了。

从业务视角来看，数据可分为事实数据、维度数据、迟到的事实（也可理解为会变化的维度），比如订单ID、商户名称，这是事实的固有属性，是不变的事实；维度数据，比如城市、配送类型等；迟到的事实，比如订单状态，在订单表中，它即是订单的一个事实指标也是订单的一个维度，但它是随时间线性渐变的，也就是说在数据抽取入仓库后，业务库还会发生变化，会出现数据一致性的问题，这时候就涉及到数据重入和更新的问题了。在外卖这个业务中，是电商+实时物流的形态，且业务周期较长，涉及到的角色主体和业务形态众多，如订单、商户、物流、用户、骑士、bd、营销、结算等等，可谓是订单类的场景中最复杂的一个。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVicz0kzcjkicPZSrYpw6QZeL6cszotnAxMywZictKbXFY9KKDpFOicsrUqw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVbho4SUiaGIBficACATBhdQmPOvBBiar4aSKJictTIV1Gb4WNRibictCnhECg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1) 

 	

面临以上复杂的场景和逻辑，如果每个ETL细节逻辑都靠编码来实现，且不说开发成本和数据解释成本，单是任务管理和代码继承及足以让开发者头疼不已。但是每一家公司的ETL基本上都是从最初的脚本+sql的形态开始，都会经历这个阶段。

在外卖ETL方向的发展可以分为这四个阶段，也是个人认为业内ETL整体发展的一个走向。从在关系型数据库上跑sql，写脚本解析数据，定时任务出报表开始，随着业务体量的发展，在计算性能和任务管理方式上很快就会遇到瓶颈，单机跑脚本会逐渐由分布式任务来代替。进入到第二阶段，map-reduce、spark之类的分布式任务和基于hadoop生态的一系列大数据解决方案实现了计算和存储的可扩展，但面临成千上万的任务，早期的ct任务靠时间约定来进行上下游约束，亦或在脚本中串行调用下游来实现，这种刀耕火种的方式在业务快速膨胀的过程中，面临的将是高昂的任务维护成本。在第三阶段，引入以调度为中心的，提供监控、任务重试机制、性能分析等一整套解决方案，来实现任务的串行依赖和并行计算的灵活组合。到这一步，就已经能解决绝大多数ETL场景的问题了。在第四阶段，做的就是ETL精细化，实现数据可用性监控，通过透明化数据生产进度，进而构建完整的数据交付体系，面临庞大的元数据体系如何降低数据解释成本，如何从表粒度的数据生产依赖，精细化到字段粒度。在这个阶段，核心就是让业务更加便捷更加自助式的使用数据，同时提升数据交付质量。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVRGElBrIJ2XIQ3H7aBbQs1uyzlzWIQ6UPiaZk5IrhPPmxZTias9tENV6w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

接下来看下百度外卖这边整体数据生产的一个数据流架构。和业内所有的数据生产场景一样，主体分为数据源、生产、仓库、数据应用四层，数据源以业务库的mysql为主，其次是hdfs或ftp的文件、kafka/nmq（百度的消息队列）、api。生产层面以ETL系统为主，主要处理业务数据，日志的基础数据生产走spark streaming。

ETL系统的执行单元是个脚本命令，由调度系统远程调起，来实现分布式，调度由任务(job)->转换(transformer)->算子(operator)的粒度进行封装，通过这种结构关系实现ETL作业的先后依赖和并行处理。在数据生产的环节中，除通用的系统监控服务外，还开发了一套数据监控服务来保证数据产出后的可用性。

数据的例行生产由底层事实表和维度表开始，基于事实和维度生产一些中间表(也称为轻度聚合表)，然后是聚合表，这也是常见的一种仓库设计。就数据的使用层面来讲，事实明细+维度的查询是能满足所有查询场景的，灵活度也最高，但随着业务体量的发展，这种基于明细的查询会带来极大的冗余查询。试想不同的业务查询同一个指标，类似的sql每个人都需要跑一遍，冗余查询引发的将是大量集群的额外开销。所以设计基于明细数据的聚合表，是一种事前计算的思路，将不同指标在不同维度下的数据提前计算好，方便业务查询的同时也统一了数据口径。在上期大数据分享中提到的kylin，其性能高效的原理和建中间表是同理，基于各种维度预建好中间表，任意维度组合的查询都能直接落在一张表的数据文件上，极大减少了hdfs文件扫描的开销。

那么中间表该怎么建，这么多的维度组合和指标，不可能面面俱到，只能用有限的集群资源去解决热点的问题。哪些查询是热点的，不可能通过逐一收集用户需求来实现，在数据使用服务上我们开发了一套adhoc（即席查询）查询服务，所有的数据出口都收敛在这里，有面向BI分析人员的sql查询UI，也有面向后端服务的同步和异步接口，通过adhoc每天记录的用户查询日志，我们能分析出哪些表和字段是查询热点，都分部在什么维度。同时通过收敛查询的出口，在adhoc这个数据出口上建立了完整的字段粒度的数据权限控制。在ETL生产的过程中，配合信号灯服务，实现了数据生产可解释；通过在数据集市中的元数据管理系统，支持面向用户的元数据和口径查询。基于对ETL配置的解析，生产了数据的血缘关系，用于描述表的上下游及字段之间的生产流转关系。有了口径和数据血缘关系的描述，就极大降低了数据解释成本。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVkvAPZlJfRtrx1U6DJ6PTq44TTWWNs1YMrTiaLcTVKxVqrjEsVEvIU2Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

基于之上的整个生产数据流，再来看下ETL内部是如何实现流转的。在前面提到我们有基于事实明细聚合查询后回填到事实宽表的查询，比如在订单维度，标记该订单是否为用户的首个完成单。同时我们有接入api推送的数据，这类事实数据入库后，在下游应用在一些实时的dashboard上。还有针对迟到的事实这种场景，需要有单个字段回溯的场景，举个例子，比如订单状态，正常的餐饮订单一般在几个小时候才完成，但是会有些预订单或物流单，在订单创建数周之后才能完成。那么基于以上场景，需要有一套支持细粒度更新的数据存储解决方案，这里就引入了一层ODS，所有的数据拼装和转换结果都先入到ODS中，再sqoop到集群，同时也会有从集群聚合查询后回填到ods的场景。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVMd5jhqjIJyR99HOHUCboNcM616FkkDn1ZOqO2UO7iacS3brVKamU3vA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

我们对ODS的定位：第一，在业务系统和仓库之间形成一个隔离层，ODS用于存放从业务系统直接抽取出来的数据，这些数据从数据结构、数据之间的逻辑关系上都与业务系统基本保持一致，因此在抽取过程中极大降低了数据转化的复杂性，而主要关注数据抽取的接口、数据量大小、抽取方式等方面的问题。第二，承担一部分业务系统的明细查询。对于明细查询的场景一般有三种，带主键的检索式查询、有时效性要求的OLTP条件检索、离线场景的明细拉取或聚合，对于前两种场景是适合放在ODS中的，面向OLAP的查询引擎无法满足其时效性。第三，支持业务的回溯更新场景。

在ODS的选型上，考虑过三种类型的存储：行增量式、列式、支持OLTP+OLAP混合的存储。这三种类型的存储，对应到这个数据场景下，需要不同的解决方案。行增量式以druid为例来讲，高性能写入，不支持更新。在外卖这边druid已经应用于日志的一些场景，那么能不能用它来解决业务数据呢，在一些特定场景下是可以的，外卖的绝大多数业务都是有主键属性的，还是拿订单状态来说，把订单的每次变更都当作一个事件，记录在druid中，下游的ETL按事件发生顺序倒叙排列取最新状态即可，但带来的问题就是会冗余存储大量过程状态，数据量级膨胀的系数取决于变化维度的枚举值个数。列式存储，基于MPP架构的以vertica和greenplum为例，其在大数据场景下的查询能力非常出色，但是针对数据回溯的场景，其满足不了批量DML操作的低延时性能要求。

我们需要的是一个融合低延迟写入和高性能分析的存储系统，结构化存储系统在hadoop生态里通常分为两类：静态数据和动态数据。静态数据，数据通常都是使用二进制格式存放到 HDFS 上面，譬如 Apache Avro，Apache Parquet。但无论是 HDFS 还是相关的系统，都是为高吞吐连续访问数据这些场景设计的，都没有很好的支持单独 record 的更新，或者是提供好的随机访问的能力。 动态数据，数据通常都是使用半结构化的方式存储，譬如 Apache HBase，Apache Cassandra。这些系统都能低延迟的读写单独的 record，但是对于一些像 SQL 分析这样需要连续大量读取数据的场景，显得有点捉紧见拙。上面的两种系统，各有自己的侧重点，一类是低延迟的随机访问特定数据，而另一类就是高吞吐的分析大量数据。简单来讲，我们需要的是一个OLTP+OLAP融合的存储系统，kudu和phoenix就是在两个特性之间找到了一个平衡点。我们线上目前用的是phoenix，它是一个基于hbase之上的组件，通过cm安装部署非常简易，因为是基于hbase的所以写入性能可扩展，而且hbase的运维比较成熟，作为一个新型引擎引入技术风险较小。Kudu作为当下的一个主流趋势，我们也搭建了测试集群正在摸索中。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIV0gBmKqFPLUwJmSmgRpO2Rf2ic6EN4EUIX11l2ibqydVKEEHNjRyL9J5A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

这里列了两张phoenix官方提供的OLAP场景下和hive、impala的性能对比图，在行数据递增的时候其查询性能耗时增长缓慢。在我们的实际应用经验中，phoenix的性能主要得益于内存的使用，当DML操作比较密集的时候同时进行复杂的DQL操作，内存吃紧的状态下其性能会显著下降，所以在外卖的使用场景中，phoenix主要用于解决行粒度更新的问题，复杂的OLAP操作在impala和hive中。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVBSa2FZ1gkCWztqMm7U0542EI6vzUjRcqxaIYRV7sOLv6v3kNyJLl2A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

前面提到了调度系统由任务(job)->转换(transformer)->算子(operator)构成，算子是最小执行单元，在ETL的作业执行中，具体的一个算子就对应一个ETL作业，调度事先将作业在线上集群初始化，通过时间触发+任务依赖的方式选择宿主机调起任务，在宿主机选择时优先选择负载最低的节点执行。任务和转换在同一层级中，依赖关系均是有向无环图，每个节点的父子节点可能都会有多个。算子作为最小执行单元，一个转换中可能会有多个算子。在调度系统中为ETL设置了多种算子类型，比如ETL算子、sqoop算子、webservice算子等，通过预定义的参数填写来配置。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVqN7f0BQ0OOOZia2RIDicloicE3cZmUyDdZib2vk2OibichC3aBcBAsUuEdQQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

下面来讲讲ETL框架的流程设计。框架最核心的设计理念，是从ETL的场景出发，可以分为两大类。这两个类别能覆盖大约90%以上的需求，一些特殊场景基于配置无法满足的，按照框架定义的标准进行编码自定义class实现（比如数据源是按业务id hash分表，比如数据主体没有业务主键）。第一类，前面提到的字段之间多对多的映射关系，从上游查询出多个字段，经过转换后插入下游某个表，如果转换逻辑可通过sql实现，对应的其实就是一个insert from select 的语法。那么在异构存储的系统中，可能会涉及到多个数据源的相互转换，无法用一个sql搞定，这个就是开放式sql做的事情，将这类的生产逻辑都转化为一张配置表，将数据抽取和装载分开执行。第一类是相对简单且常见的场景，那么针对转换逻辑无法通过sql实现的场景，我们设计了开放式index，一个针对字段粒度的etl方式。图中开发式index的配置里db¬_connect、db_connect_table、select_sql三个参数描述从哪去取源数据，db_connect_engine、fact_table、index_column描述数据往哪插入，写入哪个字段，对于需要用coding进行逻辑转换的，在handle_flag中指定自定义函数，自定义函数通过约束统一输入输出的格式来对接到上下游数据流，剩下的字段多为配置的描述性字段，如是否可回溯、指标分类、生效时间等。

基于这两类配置来看下流程图，首先初始化参数和配置，比如要生产表明为t1的表，根据参数中的设定检查对应目标表的分区是否存在，不存在就先创建，然后写入信号灯，告诉信号灯t1表开始生产了。开始进入正式的生产流程，第一步判断是否有自定义编码，有的话先加载执行，第二步查询open sql的配置，如果发现t1表的生产对应多条开放式sql的配置，则顺序执行，同时也支持在传参入口传入指定配置编号来细粒度执行。第三步查询open index，得到多条配置，以计算t1.f1 和t1.f2为例，两个字段对应不同的数据源不同的逻辑，先独立执行数据抽取，然后执行各自独立的自定义函数（如果有的话），然后再对处理结果进行merge，拼装成一个upsert语句进行插入。第四步，调用监控服务判断生产结果是否符合预期标准。第五步，写入信号灯告诉信号灯生产完成。整体ETL的生产，结合了ODS支持upsert的特性，加上open index的细粒度生产配置，实现了数据的细粒度生产与回溯；生产作业命令的参数支持按表和多字段传入，实现了计算粒度可伸缩，可整表批处理生产也可生产单个字段；同时支持ETL配置热加载，随时修改随时上线。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVoJ257r2via4Zs7fq2CV0qVzdLk11mYtwnDglDwaGV0g6TthTIcvF3Yw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

这是一个开放式sql的配置示例，截图了配置的部分字段，可以看到前两个字段描述了数据源，然后描述了数据往哪写，在转换逻辑中，查询sql的字段和写入sql的字段是一一映射的。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIV26AccyNbCltt1gzLzRqzuLian65n4n2xoLq3tVMp5bQ18VBobsziaaVQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

是一个open index中函数编码的示例，计算“代理商抽佣金额”这个指标，函数的入口统一标准格式，输入为主题数据的二维数组，数组的key为业务主键，数组第一维表示数据行，第二维表示数据列，在这个示例中需要对入参的多列分别json反解析后再进行数学逻辑运算，结果集作为新增列再填回维护数组并返回。针对这种业务逻辑复杂，需要反解自定义加密、解析复杂的数据结构的场景，我们只能编写业务代码来处理，open index支持自定义函数的设计很好的解决了这方面的扩展性。

 

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIV51dQI377Axd7JWZYdc1kFF0kD2ibMzjSfyFuJ2AQvgHyB0VucgGLLGA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

除了ETL计算的流程之外，整个ETL系统中还有重要的一环就是sqoop，我们对开源工具进行了封装，在其基础上加入了分区自动创建、并发控制、字段类型映射等功能。现在来看下sqoop的处理流程，首先在选择数据源、创建分区后，第一步先进行schema diff，做这一步主要是为了解决线上库schema变更下游未及时更改导致sqoop失败的问题，且线上业务变更数据库的操作经常在夜间进行，对大数据运维来讲同步跟进的成本很高，且夜间是数据生产的高峰期。这里我们开发了一套工具用来事前录入变更schema，通过版本控制，在检测到schema有差异时自动去加载最新的schema版本对线上hive/impala进行变更。第二步针对要同步的数据count量级，一方面是用于并发的边界处理，一方面用于同步完成后进行数据量级校验。第三步进行边界处理，即对应原生工具中的-split-by和-m参数，用于控制同步的并发度和数据倾斜。第四步进行字段映射处理，由于sqoop本质是通过一个mr任务抽取数据后生成数据文件，hive在建表关联时会存在很多字段映射的问题，比如mysql中的bigint unsigned 类型，但在hive中只有bigint类型，符号位占据了一位所以其长度不够，这种情况都需要在hive中转为string。第五步是基于配置的压缩处理，对应—compress参数。以上几步完成sqoop的命令拼装，执行后就产出数据文件了，之后释放数据生产完成信号，然后针对同步的表进行hdfs缓存池的维护，最后在impala进行元数据的compute（数据出口有hive、impala、greenplum三种引擎）。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVhx9YxCEVqXpolxksFf2s1uJOibhGdroQMzdU5fJ8pk2e7PUA5cCwndA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

 

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVugJamGQ7zxWlv9Vvf4O6BYMibBribLsN1xqCVYW6icfPXMBEAMAEAI3Yw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

以上，介绍了数据生产的核心流程，但从系统的层面来讲，数据生产完成不等于数据可用。从数据生产到交付至业务使用的场景中，经常会遇到数据生产任务执行异常、生产任务延迟、生产结果异常等，对于任务执行异常或延时，调度系统都会进行重试和报警，对于生产结果的异常，就得从数据本身去判断处理了。

 

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVfJx9A4OXVg0BEOMY0nle0uZsjdic7EIJLZzaV014dtlKI5Q9bn8y0ZA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVt9e9QbSVH58ib2lTphr8SeMVaZFCY39HwA5ugXyHUxmHxbLNQr5JzUg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

对于数据的质量问题，我们在生产任务的前后都设置了checkpoint，前置的checkpoint用于检查它所依赖的上游数据是否符合预期，后置的checkpoint用于检查自身产出的数据是否符合标准。那么具体的逻辑就是由监控服务来实现，和常见的监控服务类似，由用户自定义配置监控项，进行定时check。在数据监控服务中，和常规系统监控服务的差异在于这里的配置分为表结构监控和数据可用性监控，后者由sql实现同源或异源数据的对比、环比同比的阈值监控、空值率及类型监控，同时对于不同阈值触发不同的监控报警级别和处理措施，例如“订单量”环比波动超过20%就邮件报警，超过50%就短信电话报警，“流量转化率”为负数就停止生产任务等。监控服务同时提供api触发check的功能，由调度的webservice算子或执行脚本进行自主调用，更加灵活的支持在关键流程中设置check任务，检测到异常则终止整个生产任务流，下游生产将不会被触发直到异常点恢复。

 

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVwfaTRp1fwB5hGS3cVo1ibRRanBLty0IOdybf4nkR8RgU09Um6GiblV1A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

对于数据交付流程上来讲，用户还会关心任务的生产进度，信号灯服务从命名上就可以看出是用来释放数据生产信号的。在前面介绍ETL作业流程中就已经提到，在生产初始和结束时都会调用信号灯的接口，来释放信号。信号灯服务提供了信号写入api和信号查询api，对于下游系统来讲只需要监测数据可用信号即可，在服务层面实现了通过api解耦。在数据的上下游关系中，业内常见的依赖都是整表依赖，整表依赖的最大痛点在于对于宽表来说，下游的生产开始时间取决于上游最后一个字段的生产结束时间，而依赖关系中可能只是使用到了上游的部分字段。在前面介绍的ETL生产流程中，我们的ETL系统已经支持到了字段粒度的生产，那么对于数据交付来讲，信号灯的信号释放表现在字段粒度，就可以做到字段粒度的作业流依赖。

以上，构建了完整的数据交付体系

 

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVHvB71aDC3UImvWSwARqUtVlJxdq4m9jIibnrqZ0U8HFgFbIKnIKNiaiaA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVYUfp9cxwqhpY8Wd3r0lSGF2WsRG0ibXbsTktVehfMMySXUP8Rz4mnqw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pJE78Atczrnep17bMEVRX5NgjEvqvvIVbs5VjFa2ic8W4ufrroewsVic0McXJ4icUwqWFdibuULtm2qJs9bXnszBkw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

在数据使用层面，数据的可解释性是所有大数据团队的一个痛点。数据可解释主要包含两个方面：数据口径和数据依赖关系。ETL开发者经常面临的一个问题就是要向数据使用方解释你的数据是如何生产出来的。在业内常见的生产依赖关系都是数据表和生产任务之间的依赖，对于生产任务具体用到了上游数据表中的哪些字段只有在编码逻辑中才能体现，是无法暴露给数据使用方的。由于我们的ETL系统绝大多数生产逻辑都是基于配置的，因此通过配置文件即可解析出数据上下游的生产依赖关系 - -即数据血缘关系。在调度配置的任务中，一个任务对应一张表的生产，通过血缘关系可以清晰看到这个任务在数据层面的依赖关系，做到数据可解释、生产过程可解释。