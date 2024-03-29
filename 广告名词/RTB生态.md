![[Pasted image 20220312190554.png]]

RTB模式就是将投放平台从与流量聚合的利益共同体中分离出来，使得投放平台更多地和广告主的利益结合在一起。具体的利益结合方式主要有三种，**对应DSP的三种盈利模式：套利模式，服务费模式，消耗分成模式**。后面会一一介绍。

# DSP盈利模式

## 套利模式
广告主和DSP签订某个行为数的购买价格，例如每个点击1元钱。然后由DSP通过ADX采买流量，DSP会通过技术手段将每个点击控制在例如0.8元钱，这时候DSP就可以赚0.2元的差价。DSP的技术越强，相同的量的情况下，就可以以更低的成本从ADX里购买到点击，例如降低到0.6元，那么就可以获得更高的利润。

广告主和DSP以点击数或者展现结算，但是也可以实现和大媒体平台类似的oCPX模式，即广告主在DSP平台上也可以按激活或者付费来出价。例如广告主和DSP以一个点击1元钱结算，但是同时对每个激活出价60元。和之前介绍的oCPX一样，DSP会根据预估的p(c->a)将激活出价转换到点击出价，再根据p(m->c)转化到eCPM。同时通过成本控制，将点击成本控制在比如说0.8元，那么DSP就可以每个点击赚0.2元，这也为什么叫套利的原因。DSP对点击率预估地越准，就可以用越低的成本买到同样质量的点击，从而盈利也越高。另外，也需要将广告主的激活成本控制在60元以下。

![[Pasted image 20220313001022.png]]

## 广告主自营DSP/服务费模式
还有些DSP采用固定服务费的方式，即广告主交一笔钱，DSP就专心为广告主服务，DSP的利润和任何行为数都没有直接关系。这种方式从利益关系的角度非常接近广告主自建DSP的模式，只是和其他模式一样不一定能用上广告主所有的私有数据。

![[Pasted image 20220313001522.png]]

## 消耗分成模式
还有的DSP采取消耗分成的模式，即帮广告主花了多少预算，按一个约定的比例分成。

![[Pasted image 20220313001803.png]]

在消耗分成模式下，DSP的收入和广告主的消耗成正比，也就是和媒体的收入成正比，让DSP的利益反而和媒体绑定了。不过这种模式仍然存在是因为这张表分析的只是短期的利益关系，长期而言，如果广告主ROI下降，就会终止和DSP的合作，换其他的DSP。而DSP作为比联盟更需要代表广告主的角色，不太会做短期伤害广告主利益的事情。

