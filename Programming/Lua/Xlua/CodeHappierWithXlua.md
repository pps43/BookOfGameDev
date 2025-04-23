#愉快地在Unity里与Xlua玩耍

> Xlua原是腾讯内部项目，后来开源，相比于之前市面上的Unity+Lua热更新解决方案，有着自己的优势。要注意的是，Xlua不仅用于Unity，只是我们这里重点讨论其在Unity中的使用。
开源项目地址： https://github.com/Tencent/xLua （作者：车雄生）
由于Unity不能原生支持Lua，在使用Lua进行开发时，编写和调试难度比c#要大，为此有必要使用辅助工具，加速开发流程。这里简要地介绍一些个人觉得不错的辅助工具。

## I. 基本开发环境
- Visual Studio 2015 (Community)
- [BabeLua](http://babelua.codeplex.com/)。是一个VS插件，安装和下载都比较简便。
- Lua 5.3.3

### 代码自动补全
- 首先，下载这个[压缩包](http://url.cn/48TJGSW)
- 参照压缩包里的 readme.txt 操作即可，效果不错。如果是新手，最好参考以下具体说明。
- （1）`ExportLuaSyntax.cs`中报错的部分，直接注释掉即可。但从中可以发现可以自定义一些需要导出给BabeLua用来给出语法提示的类名。
- （2）关于新建Lua工程的说明。
由于unity识别不了lua，所以不会自动将其添加到我们的c#工程中，需要在同一个解决方案里自己手动新建一个lua工程，来管理我们的lua代码。创建新lua工程填写参数如下图所示
![](/resources/babelua_new_proj.JPG)
- `Lua scripts folder`的含义就是你存放自己的lua逻辑代码的地方。
- `Working path`一般和上面填的一致
- `Lua project name`是lua工程显示在VS里的名字而已。
- `Command line`不填也可以，除非你想从命令行执行你的lua代码。
新建工程后，右键`Add item`添加一个`hello.lua`，这就可以了。
- （3）在Unity中的Xlua菜单下选择`ExportluaSyntax`，重启VS，试着在hello.lua中打出`xlua.`，应当有完善的提示了。


