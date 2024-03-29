# 帧同步

## 一、概念澄清
讨论帧同步时，往往不同的人对于一些细节有不同的见解而争执不下，这往往是因为“帧同步”这个名词有歧义，而且并不能概括这套方案的特征。知乎有个帖子对此进行了细致的讨论（[这里](https://zhuanlan.zhihu.com/p/32843758)），结论如图：

![](/resources/frameSyncConcept.jpg)

本文所说的帧同步，其实是上图中“frame lockstep sync”，状态同步是“status sync”。

## 二、王者荣耀团队分享
### （一）帧同步和状态同步的对比
#### 状态同步的方案特点
- 优点
  - **安全**（因为有关键逻辑都在服务器上）
  - **网络要求低**
  - **断线重回快**（因为有状态快照）
  - **客户端性能优化**（视野之外逻辑可以不要的，反正服务器在算全局的）。所以千人大战这种一定是C/S架构的。
- 缺点
  - **开发慢**（因为要联调）
  - **录像回放实现较难**
  - **打击感难调**
  - **耗流量**

#### 帧同步的方案特点
- 优点和缺点恰好是状态同步方案的互补。其中外挂是一个重点，而断线重回时间长是一个难点，不能胜任千人大战是无法突破的瓶颈（因为所有东西都要放在客户端算）。

此外，帧同步常用到buffer，其延迟比状态同步是要大的。

#### 王者荣耀团队是如何取舍的
当时主要是开发效率的角度考虑，还有打击感。省流量和方便做回放是附带的好处。

###（二）帧同步实现要点
在实践帧同步的时候，有以下几个方面需要把控：
- 相同初始状态
- 一致的输入
- 一致的代码执行流程
- 一致的数学库（随机数、去浮点改用整数）
- 第三方库改造
- 开发人员写逻辑时要去除“我”这个玩家角色的概念
- 开发人员要能在发生不同步时定位问题


###（三）帧同步会遇到的问题和解决方法
#### 延迟和卡顿
延迟和卡顿是一对 trade-off，取决于buffer的大小。

首先说卡顿。这里不是指渲染性能或逻辑处理上的瓶颈（事实上画面卡顿的原因很多，后面会补充），而是特指严格按帧进行表现时，会由于网络延迟时大时小，造成角色移动不平滑等现象。

为了解决网络延迟抖动的问题，帧同步方案设计了一个buffer，收到网络的逻辑帧后，会先在本地缓存若干帧，按照固定延迟去播放，这样逻辑执行就比较平稳看上去不会一顿一顿的。但很明显这样操作和反馈都比较滞后。这就是延迟问题。

#### 解决方法：
对于延迟，彻底废除buffer，并且用UDP。对于UDP丢包问题，上行采用重发3次，下行采用根据网络情况在2次到9次范围内调整。对于关键信息（比如结算），采用RUDP保证可靠性。

对于由此产生的卡顿，采用逻辑和表现分离，逻辑不平滑，但表现上（单纯是皮肤）通过插值去追逻辑影子对象。

这里重点提一下 **逻辑和表现分离** 是怎么实现。

首先分为逻辑空间和表现空间。

逻辑空间严格按照帧同步的要求进行计算（比如碰撞啊技能释放啊都是在这里，不能用浮点数）。表现空间其实就是gameObject相关了（比如gameObject的位置啊、动画啊）。这样的话，就不能利用Unity自带的物理碰撞了，因为那些Collider是作为组件和gameObject绑死的，要自己去计算碰撞盒。

逻辑对象有ID，表现对象有VID，并且二者互相持有对方的引用。这样，表现对象就可以通过这个引用/ID去获取对应逻辑对象的位置、方向等信息，来追赶这个目标点。

追赶主要是插值（内插用于平滑，外插用于估计）。当目标点出现拐点的时候，估计势必出现偏差，这时要能圆滑地跟过来（偏差太大还是会偷偷强制滑过来）。这里有一些参数需要打磨。


#### 外挂
由于帧同步的逻辑计算在客户端，所以一旦被破解，就能拿到全局的数据从而开发外挂。最常见的是“视野挂”。
解决方法是：
- 对于PVP，采用多人关键数据hash校验判定。不过1v1没法解决。
- 不用c#写游戏逻辑，用c++写一个gameCore（后面会细说）
- 游戏外打击外挂

> 王者荣耀的 `gameCore` 是把逻辑部分剥离出来，完全用c++实现。这样做的初衷是提高性能、防止核心代码被破解。目前gameCore相当于客户端都内置了一个服务器，收到网络包后进行处理，再指挥表现层（客户端角色）渲染。这不仅是更优、更清晰的架构，还为更多问题的解决提供了基础。

#### 优化

这里主要是优化开发过程中遇到的不同步问题的bug定位效率。通常是代码写的有问题导致不同步，解决方法是：代码染色辅以体验服小范围测试。通过染色能够回溯到具体有问题的代码。

另外，CPU的负载均衡也是要做的，这有助于最终画面更流畅（之前卡顿主要是网络同步引起，但这里可以锦上添花），FPS更稳定。

#### 断线重回
帧同步的断线重回，朴素地看，只有从第一帧开始拉取并加速播放到当前帧才行。这样时间太久。













