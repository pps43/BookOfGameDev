TODO:

## 帧同步、状态同步及其在游戏中的实现。

帧同步减轻了服务器的压力，将各客户端的输入综合，然后广播下发，各客户端本地做表现。因此各客户端的随机种子要一样，还有防止浮点数在不同cpu上计算差异的积累。

资料：
http://blog.csdn.net/xufeng0991/article/details/51363790

http://www.skywind.me/blog/archives/131

帧同步-gad问答
动画组件还能用吗
http://gad.qq.com/wenda/detail/10728
http://gad.qq.com/article/detail/22336  写的挺好
http://gad.qq.com/wenda/activity/10016?sessionUserType=BFT.PARAMS.23674
http://gad.qq.com/article/detail/28435  unity上海

外网分享的王者荣耀技术方案
http://youxiputao.com/articles/12939
http://youxiputao.com/articles/11842

内部讲座的王者荣耀技术方案（腾讯微云）