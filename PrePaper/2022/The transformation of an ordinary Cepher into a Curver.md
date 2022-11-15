# 写在前面

首先，需要说明的是，我也一直认为Ceph是非常优秀的开源存储软件，想起来我接触Ceph也有好几年之久了，Ceph除了为我带来了知识上的收获，还为我带来了生活上的温饱，所以我对Ceph充满了感恩与Respect。

其次，尽管已经在存储这块领域涉猎了好几年，但我依然只算得上是一个存储新人，因为存储涉及到的领域是既宽广又深入的。存储的数据对于任何一家公司来说都是无价的资产，所以作为一个存储人有必要永远保持一颗敬畏之心，敬畏自己敲下的每一行代码，敬畏自己敲下的每一个线上命令。

存储抑或云计算的圈子其实很小，所以非常荣幸借Curve这样一个平台，能与那些既陌生又熟悉的存储人一起交流与学习。

# 我在网易做Ceph

好风凭借力，存储其实很难作为一款独立的产品提供给用户使用，也正因为如此，Ceph先后借着OpenStack以及k8s这两股东风获得了快速的发展，逐渐成为了开源领域的明星产品。

网易这边差不多是15年开始关注Ceph的，我本人的话是16年加入网易的，所以基本上算是一个早期的阶段。最开始我们是基于OpenStack使用Ceph块存储提供虚拟机，15年左右其实也是移动互联网蓬勃发展的好时机，那时候网易集团先后孵化了网易云音乐，网易严选，网易考拉等业务，借着这些业务的发展，我们使用Ceph的体量也得到了飞速的发展，比如15年的时候我们可能整个也就几百个osd，16年的时候已经有5000+个osd了，17年的话已经有15000+个osd了，到了18年已经有30000+个osd了，同时最大规模的单集群达到了4000+个osd。

然后在18年左右的时候，随着k8s的应用以及普及，公司内外的很多业务都开始拥抱容器化了，那么在当时的场景下，Ceph文件存储便逐渐有超越Ceph块存储之势。我们也在这一阶段开始把Ceph的版本切换到了L版本。

在从事Ceph这段时间里，我虽然做了一些事情，但坦率地讲，我做的事情其实并不多，以前跟圈里从事Ceph的小伙伴们聊天，大家经常会开玩笑说我们其实不算是研发(懂的都懂，这里为了不招职业黑，原话就不贴了，哈哈)。

因为所做的事情确实并不多，所以几句话就足以总结我这几年做过的事情：维护了几个还算大的Ceph集群；分析并优化了几项Ceph的性能；优化了几项Ceph的抖动问题；完善了Ceph的运维体系；向社区提交了几个简单的pr。奥，还有，拯救了几次线上集群，或许，这几年里，也只有少数的这几个时刻，让我觉得自己像是一个超级马里奥。



# Curve：缘起

- 缘起1：来自业务的无数次diss


对于大规模线上集群来说的话，坏盘，慢盘，raid卡故障或者是服务器故障等异常场景是很频繁的。但是因为Ceph三副本的强一致性协议，那么需要三副本均写完了，才能返回给业务写成功。所以这时，但凡有一个点异常了，写io便迟迟无法返回给客户端；更为要紧的是，ceph的池化概念导致了属于该池子的所有业务卷都会受影响。

所以往往就是Ceph线上但凡有一处异常，便会导致大面积的用户卷io发生卡顿。由于像网易云音乐以及电商这些业务的用户对卡顿是非常敏感的，所以每到这时业务这个甲方爸爸便会找到我们，要求我们给出解决方案。


- 缘起2：降本增效

众所周知，Ceph的均衡性现在还不是特别好(当然，现在有自动均衡，但是我理解线上应该不敢轻易上这功能；至于pg upmap的话运维就比较麻烦了)。

对于线上Ceph集群来说，不同硬盘使用率差别可能有百分之二三十。但是问题在于，Ceph集群存在着短板效应，尤其是Ceph的老版本，一个osd full了，整个集群就不可服务io了；后续版本的话一个osd满了，它对应的pool所属的所有卷便都不能服务io了。

所以，线上的现实就是，Ceph集群的平均使用率到了百分之七十多，大家就会赶紧想着扩容，要不然万一一个osd写满了，那可能就要full跑路了。

所以这个问题势必就带来了成本上的大幅上升，明明才使用百分之七十多，却要着急扩容。如果你要扩容，那势必会增加服务器，增加硬盘，增加网络以及机房托管费等层层费用开销。

- 缘起3：性能

当前来说的话，是希望我们的存储能够支撑数据库场景。

Ceph的性能优化也一直是社区的一个方向，但是基于Ceph的架构和功能比较复杂，所以这个优化过程是一个长期的过程。

基于这几个主要方面的原因，我们开始了Curve的研发。Curve在稳定性方面，线上大规模稳定运行了两年多，无任何SLA损失；在成本方面，由于Curve出色的均衡性，集群使用率可以接近百分之百；在性能方面，Curve的性能在大多数场景下应该是优于Ceph的，具体情况可以参见curve github项目主页：https://github.com/opencurve/curve

# Curve：现状，未来

 Curve项目2020年正式开源，然后今年6月份，Curve被正式接纳为 CNCF沙箱(Sandbox)项目。
 
 Curve当前提供块存储和文件存储能力，Curve的块存储已经在线上大规模稳定运行两三年了，Curve文件存储第一个正式稳定版本于今年7月发布，目前正在持续迭代中。
 
 对于块存储，稳定性和可用性已经有了比较好的保证，并且当前也有着不错的性能，但是我们有着更高的目标，我们当前正在与云原生数据库团队合作，期望打造更加高性能的可以支撑数据库的块存储。
 
 对于文件存储，我们期望打造性能与成本可兼得的云原生文件系统。因为我们文件系统的数据后端既可以对接Curve的块存储，也可以对接世面上的对象存储系统(比如aws s3,nos,minio等等)。对于追求性能的用户，数据后端可以直接使用Curve块存储；对于追求成本的用户，数据后端可以直接只用对象存储；另外，我们也可以把Curve块存储作为对象存储的缓存层。
 
# Curve推荐词

如果要我给大家推荐Curve找几个关键词的话，我想我会用上以下几个关键词：稳定性；低成本；高性能；国产；残缺美。


Curve作为一款年轻的开源存储系统，与Ceph相比肯定有很多不足之处。但是Curve相比Ceph，在某些方面也有着一定的优势，所以如果这些优势正好也是您所关注的，那么我们非常荣幸，能有机会向您推荐Curve。

- 稳定性

如前文所述，Ceph在各种硬件异常导致的单副本产生故障时，会造成大面积的SLA损失，Curve则不会有任何SLA损失。

- 成本

如前文所述，Curve相较于Ceph更优的均衡性，可以降低扩容要求，减少大量硬件成本。

- 性能

如前文所述，Curve的性能在大多数场景下应该是优于Ceph的。

- 国产开源

Curve社区的发起者和参与者目前主要都在国内，能形成更好的交流氛围与途径。

当前国际形势下，建设国产自主可控体系、大力发展信创产业已成为“数字基建”的目标，Curve 获得国家工业信息安全发展研究中心的信创认证，验证了 Curve 与信创供应链中其他产品的高度兼容适配。

此外，在数据库领域，Curve已经成为PolarDB for PostgreSQL的官方生态合作伙伴，为后者提供分布式共享存储。

- 残缺美

Ceph是一款优秀的开源存储软件，发展了十几年，Ceph的功能已经非常完善，想要参与社区或者精通Ceph是一件非常难的事情。而Curve才开源两年，尤其是Curve文件存储，第一个稳定版本才正式发布几个月，正是因为Curve的不完善，Curve社区接下来才有更多的事情要做，所以我们也更加渴望与期盼更多的朋友跟我们一起来发展，壮大这个社区，让我们都能成为国内开源存储的一份子。

# 写在最后

写这篇文章其实很惶恐，但若能藉此让朋友们知道我现在正在做什么，甚或能吸引三两志同道合的知己，一起来完善开源Curve社区这么一件有趣且有价值的的事情，那便是我莫大的荣幸了。