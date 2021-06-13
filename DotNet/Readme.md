# c#和.NET学习资料

笔者是从Unity接触c#的，这样既有好处也有局限性。好处是趣味性，局限性是由于Unity采用的c#运行时是免费的`Mono`，对c#的新特性和.NET核心机制并不关注。后来进入微软后，有机会从非游戏的角度了解庞大的.NET生态和远比`Mono`先进的c#运行时`CLR`。同时.NET也在经历着变革与新生：由于优异的跨平台特性和运行效率，`.Net Core`将替代`.Net Framework`，开启新的征程。

这里收集了一些不错的学习资料，不仅适用于c#，还能深化对**类型系统设计、内存管理、性能优化**的理解。

## 书籍
- 《C# in Depth》，[官网链接](https://csharpindepth.com/)，第四版更新到了c#7和部分c#8的特性，语言方面看此书基本足够。
- 《CLR via C#》，第四版于2015年出版，虽然基于.Net Framework 4.5，但依旧是当前市面上对CLR介绍最详实的一部必读好书。
- 《The Book of The Runtime》，[Github链接](https://github.com/dotnet/coreclr/tree/master/Documentation/botr)。这里收集了微软C#和.NET运行时设计团队的若干文章，可以作为阅读《CLR via C#》的补充。
- 《Pro .NET Memory Management》，[官网链接](https://prodotnetmemory.com/)，从内存管理和性能优化角度，介绍.NET的实现细节和实用技巧，这本书得到了微软C#设计团队的推荐。作者还贴心地公开了两张的海报，一张介绍[.NET内存排布](https://prodotnetmemory.com/data/netmemoryposter.pdf)，一张介绍[.NET GC](https://prodotnetmemory.com/data/netmemoryposter_threads.pdf)
- 《The Garbage Collection Handbook》，对于GC这个庞大的话题，上面的书籍并未系统性的介绍，而这本书由浅入深地介绍了GC这个庞大的话题，乃经典必读。

## 其他