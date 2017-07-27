# UGUI 渲染系统

本篇从几个角度介绍UGUI的渲染方面的概念、常用技巧和优化方法。

# 渲染器

UGUI的渲染器是 **Canvas Renderer**（下文简称为CR），首先来看看其与同样是渲染2D物体的 **Sprite Renderer**（下文简称为SR） 的区别和联系（[原文链接](https://rubentorresbonet.wordpress.com/2016/05/26/unity-sprites-spriterenderer-vs-canvasrenderer-ui-image/)）。

### 相似点：
- 都有一个渲染队列来处理透明物体，从后往前渲染。
- 都可以通过图集（Atlas）来合并渲染批次，降低Drawcall。

### 不同点：
- CR与RectTransform配合，因此可以有专门的布局适配，但必须在Canvas中使用，常用于UI。SR与Tranform配合，没有适配，常用于GamePlay。
- CR是基于矩形分割的三角网格，一张网格**最少**有两个三角形，透明部分也占用空间。SR的三角网格比较复杂，不过也是自动生成的，能剔除大部分透明部分。

>  之所以使用CR渲染一张图最少有2个三角形，是因为仅在`ImageType`设置为simple时成立。设置为sliced（九宫格），将会有18个三角形，设置为tiled可能更多。对于sprite renderer，若非透明的部分不连通，则为每个非连通部分各自生成网格。在Unity编辑器中观察三角形网格可以在scene中选择`Shaded Wireframe`。

### 小结
可以看到，三角形数和网格面积是一对“trade-off”。

网格三角形数和顶点数成正比，影响的是GPU的几何运算量。网格所占面积影响的是GPU的片元计算量。

而对于现代对于移动处理平台，**对顶点操作的代价要显著小于对片元操作的代价**。尤其是处理半透明图片的重叠时，需要反复对同一个像素位置进行渲染，产生overdraw问题。

SR用更复杂的顶点运算为代价，剔除了大量透明区域，减少了overdraw，换取了更少的片元处理，节省了GPU带宽。而且，变化的SR不会像CR那样引起整个canvas去rebuild。本人认为，在显示不需要使用布局功能的2D物体时，可优先考虑用SR而不是通通放到Canvas下做成UI，这样既可以减轻overdraw、也能防止潜在的过量rebuild。


### 补充一下
上面提到了SR的网格是自动生成的。其实，Unity自带的SR网格生成算法并不好，更好的算法[见这里](https://www.codeandweb.com/texturepacker)。

另外，渲染3D物体用的是MeshRenderer（简称MR），它和SR的区别在哪里呢？MR可以接受光照并且投射阴影。而SR不能原生支持这一点（ https://forum.unity3d.com/threads/why-cant-sprites-gameobjects-cast-shadows.215461/ ）。



# 渲染层级控制
（待补充）


# 

