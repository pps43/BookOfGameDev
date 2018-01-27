# 帧同步。

帧同步减轻了服务器的压力，将各客户端的输入综合，然后广播下发，各客户端本地做表现。因此各客户端的随机种子要一样，还有防止浮点数在不同cpu上计算差异的积累。

资料：
http://blog.csdn.net/xufeng0991/article/details/51363790

http://www.skywind.me/blog/archives/131
