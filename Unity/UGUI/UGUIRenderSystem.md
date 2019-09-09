# 浅析UGUI的渲染机制

本篇简要介绍UGUI的渲染方面的概念、常用技巧和优化方法。可能写的不太系统，而意在写一些重点。

# 渲染器

UGUI的渲染器是 **Canvas Renderer**（下文简称为CR），首先来看看其与同样是渲染2D物体的 **Sprite Renderer**（下文简称为SR） 的区别和联系（[原文链接](https://rubentorresbonet.wordpress.com/2016/05/26/unity-sprites-spriterenderer-vs-canvasrenderer-ui-image/)）。

#### 相似点
- 都有一个渲染队列来处理透明物体，从后往前渲染。
- 都可以通过图集（Atlas）来合并渲染批次，降低Drawcall。

#### 不同点
- CR与RectTransform配合，因此可以有专门的布局适配，但必须在Canvas中使用，常用于UI。SR与Tranform配合，没有适配，常用于GamePlay。
- CR是基于矩形分割的三角网格，一张网格**最少**有两个三角形，透明部分也占用空间。SR的三角网格比较复杂，不过也是自动生成的，能剔除大部分透明部分。

>  之所以使用CR渲染一张图最少有2个三角形，是因为仅在`ImageType`设置为simple时成立。设置为sliced（九宫格），将会有18个三角形，设置为tiled可能更多。

![非连通sprite mesh](/assets/spriterenderMesh.png)

> 对于sprite renderer，若非透明的部分不连通，则为每个非连通部分各自生成网格。在Unity编辑器中观察三角形网格可以在scene中选择`Shaded Wireframe`。



#### 小结
可以看到，三角形数和网格面积是一对“trade-off”。

网格三角形数和顶点数成正比，影响的是GPU的几何运算量。网格所占面积影响的是GPU的片元计算量。

而对于现代对于移动处理平台，**对顶点操作的代价要显著小于对片元操作的代价**。尤其是处理半透明图片的重叠时，需要反复对同一个像素位置进行渲染，产生overdraw问题。

SR用更复杂的顶点运算为代价，剔除了大量透明区域，减少了overdraw，换取了更少的片元处理，节省了GPU带宽。而且，变化的SR不会像CR那样引起整个canvas去rebuild。本人认为，在显示不需要使用布局功能的2D物体时，可优先考虑用SR而不是通通放到Canvas下做成UI，这样既可以减轻overdraw、也能防止潜在的过量rebuild。


#### 补充
上面提到了SR的网格是自动生成的。其实，Unity自带的SR网格生成算法并不好，更好的算法[见这里](https://www.codeandweb.com/texturepacker)。

另外，渲染3D物体用的是MeshRenderer（简称MR），它和SR的区别在哪里呢？MR可以接受光照并且投射阴影。而SR不能原生支持这一点（ https://forum.unity3d.com/threads/why-cant-sprites-gameobjects-cast-shadows.215461/ ）。






# 渲染层级（TODO）
我们说的渲染层级高，是指它盖在其他物体的上面，也就是最后被渲染（颜色混合）的那个。

渲染的层级控制本身也是分层的。其从高到低分为几个层次：
（1）相机的layer和depth
（2）画布的layer和order
（3）物体的hierachy关系。

越高的层次，控制力就越大。下面，我们具体说明每个层次的使用。

## 相机层级

![](/assets/unityCamera0.png)

上图的红框中，`CullingMask`可以决定该相机能看到什么`Layer`的物体（`Layer`在Inspector窗口的右上角设置，可以自定义）。

`Depth`越高的相机，其视野内能看到的所有物体的渲染层级都更高（无视其他低层次的规则）

至于`ViewPort Rect`，可以实现同一个屏幕内显示多个相机视野的功能，通过设置起始点和宽高，可以实现类似监控台多个监视器的效果。下面贴一张图做例子。

![](/assets/unityCamera1.png)

## 画布层级

## 物体层级
父子parent在下面
兄弟sibling

# 渲染流程

这里重点说CPU的部分。

## （一）rebatch
### 什么是rebatch

将mesh转化为渲染指令的过程叫做rebatch，也叫batch build。Unity中，主要是由canvas相关组件完成rebath，大致步骤是：

> `geometry` -> canvas renderer -> `meshes` -> canvas -> `rendering command` -> unity graphic pipeline。

### rebatch何时触发
当一个canvas中包含的mesh发生改变时就触发，例如SetActive、transform的改变、 颜色改变、文本内容改变等等。注意，这些变化影响的范围是以一个个canvas为界限的，即**其他canvas、子canvas内的变化与当前canvas无关**（后面会说到，这也是用多个canvas或子canvas能减少rebatch和rebuild的原因）。

若canavs内含有需要rebatch的物体，则该canvas会被标记为`dirty`，供后续流程使用。

### rebatch很慢吗
虽然rebatch这一步是多线程的，但不必要的rebatch，会给后续步骤带来很大负担。rebatch本身对计算力的消耗体现在：对meshes按照深度和重叠情况排序、共享材质的检测等。

## （二）rebuild
### 什么是rebuild
布局和网格被重新计算的过程叫做rebuild。Unity中，是指c#中Graphic组件的重新计算（可以理解为rebatch的后续）。大致步骤是：每一帧都会触发WillRenderCanvases事件，然后由CanvasUpdateRegistry响应并执行PerformUpdate。

PerformUpdate分三步：
1. Dirty `Layout components` rebuild ,有一个根据Hierachy排序过程
2. Masks Clipping component
3. Dirty `Graphic components` rebuild, 不需要排序

### rebuild很慢吗
Unity官方没说用的是多线程还是单线程处理。不过可以肯定的是，因为Layout components需要排序，所以要尽量少用Layout components，而用rectranform的锚点来布局。


## （三）drawcall
### 什么是drawcall
CPU和GPU之间的通信内容主要有两类：数据和命令。其中，命令又分为两种：
（1）设置渲染状态（2）drawcall。

渲染状态主要包含所用shader、材质（包含纹理、uv信息、光照等）。
> 这里补充一点：Image组件绑定的材质不要设置为`none`，否则unity会采用默认的材质。该材质的shader过于复杂，会降低性能。没啥需求就选成`sprite-default`。

drawcall的确切含义是：**在渲染的第一阶段，CPU准备好数据和渲染状态后，真正向GPU发送的一次渲染请求：开始渲染一个或一批图元**。

### 为什么drawcall太高不好
drawcall太高，意味着有频繁地改变渲染状态，这是很慢的操作（相对于GPU执行其内部的渲染流水线来说）。当存在两个相同渲染状态、仅仅是顶点数据不同的图元时，我们应当尽量将其一起渲染，这样就减少了一次drawcall。而GPU的空闲时间也会减少，最终完成渲染整个画面需要的总时间就会减少，帧数才能提高。

如果场景中drawcall太高，那么得从两个方面入手：检查hierachy的合理性；使用图集。具体方法这里后面有文章会专门介绍。

但要记住一点，虽然通常我们都在尽量降低drawcall，但并不是越低越好。所以不要一味追求低drawcall而牺牲了其他方面的性能。


## （四）fillRate
这个是显卡里面的概念，这里单独拎出来，是因为overdraw导致的fillRate过大问题通常是移动设备GPU渲染的瓶颈（而不是顶点数太多）。关于这块的优化会单独抽一篇文章去说，这里给出fillrate的公式。

> fillRate = 像素数 x overdraw x shader复杂度

另外，fillrate的概念可细分为pixel fillrate和texture fillrate。

####pixel fillrate
显卡每秒可渲染到屏幕上的像素点数量。有点像带宽的概念。在片元着色性能成为显卡性能评估标准之前，像素填充率一直是最准确的标准之一。

#### texture fillrate
显卡每秒可渲染到屏幕上的纹理化了的像素点数量。摘一段wiki：
> To render a 3D scene, textures are mapped over the top of polygon meshes. This is called texture mapping and is accomplished by texture mapping units (TMUs) on the videocard. Texture fill rate is a measure of the speed with which a particular card can perform texture mapping.




# 优化技巧举例
这里是指写几个简单明了的技巧，UGUI还有诸多方面的全面优化参见后文。

## （一）隐藏UI的正确方法
如果用setactive（false），会导致所在canvas的VBO数据失效。所以再次setactive（true）的时候，会导致整个canvas去`rebuild`和`rebatch`，从而对CPU造成负担。

推荐方法：**多使用sub-Canvas**（注意不仅要添加`Canvas`组件，还有`Graphic Raycaster`组件），隐藏目标采用将对应的`CanvasRenderer.enabled = false`。

但注意挂在上面的脚本还是在运行的，而且虽然看不见了，还是能被Graphic Raycast检测到。如果这样会对逻辑行为造成影响，那么应该自己实现一个manager去管理CanvasRenderer.enabled 下的脚本行为。

[参考资料](https://unity3d.com/learn/tutorials/topics/best-practices/other-ui-optimization-techniques-and-tips#disabling-canvas-renderers)

## （二）降低overdraw的技巧
首先观察当前场景的overdraw：unity编辑器中scene视图中选择`OverDraw`（默认是`shaderd`），越亮的区域，overdraw越高。

![](/assets/overdraw.png)

对于降低overdraw，有这么几种办法：
（1）全屏遮挡的情况，则为被遮挡的canvas添加`CanvasGroup`组件，然后在被挡住时将其alpha值设为0，就不会传给GPU渲染了。Canvas Group还有一些其他设置，意味着该节点以及所有的子节点都可以统一地更改一些行为。

![](/assets/canvasGroup.PNG)

`Blocks Raycasts`是一个很有用的设置，取消勾选，则意味着该canvas里面的UI都不会接收点击等事件了。

`Ignore Parent Group`如果勾选，则表明该canvas无视上层CanvasGroup的设置。

（2）非全屏遮挡，这就要从美术资源上想办法。