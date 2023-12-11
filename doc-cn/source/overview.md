概览
===========

本节解释了该体素引擎的主要概念, 以及实现相关功能的各个组成部分

什么是体素
------------------

![Cubes and marching cubes](images/cubes_and_marching_cubes.webp)

"体素(voxel)"是"体积图元(volumetric picture element)"的简称, 类似于"像素(pixel)"和"图元(picture element)". 它们组成体积, 而不是简单的 2D 图像, 它允许在 3 维中制作 3D 地形或模型. 与传统基于多边形建模这类只表示物体表面的方法不同, 体素建模也能表示物体的内部, 包括空间的每一个点.

在此引擎中, 体素是空间中保存某些值的特定点. 这些值可以是:

- 体素的类型
- 体素的密度 (或者带符号距离)
- 颜色
- 材质
- 等等...

体素可以通过在网格上手动设定值获得或者使用分形噪声或符号距离场公式等程序化规则在任意地方生成.


> 注意
> 尽管该引擎最初是作为生成地形的一种方式, 但这并不是您可以用它做的唯一事情. 因此, 您经常会在本文档中找到“体积volume”一词, 而不是“地形terrain”, 以指定由体素组成的对象.



体素生成
------------------

体素包含三个维度, 因此和图片不同, 随着体积的增大存储体素会消耗更多的内存. 这就是考虑体素的程序源非常重要的原因, 因为它们不消耗任何内存并且可以以任何分辨率访问.

这部分任务通过派生[VoxelGenerator](api/VoxelGenerator.md)来实现. 他们的目标是在空间中的特定点或者一个完整被定义的体积中生成体素数据. 生成模型、地形或星球的方法有无数种, 这些在本节中不做详细介绍.

生成器包含以下类型:

- [VoxelGeneratorNoise2D](api/VoxelGeneratorNoise2D.md): 使用 2D 分形噪声生成高度图
- [VoxelGeneratorNoise](api/VoxelGeneratorNoise.md): 使用 3D 噪声生成一个“海绵状”世界, 形成大型洞穴和悬垂物, 并进行修改, 使大部分地形向下, 仅在高于某个高度时变成空气
- [VoxelGeneratorFlat](api/VoxelGeneratorFlat.md): 生成一个简单的平面
- [VoxelGeneratorGraph](api/VoxelGeneratorGraph.md): 允许使用图形（如 3D VisualShader）组合各种操作, 以比其他简单生成器更灵活地生成体积
- [VoxelGeneratorScript](api/VoxelGeneratorScript.md): 允许使用脚本实现生成逻辑. 可能会比其他选项慢, 除非您使用 C# 或 GDNative.
- 其他


体素存储

---------------

图块
![Block map storage schema](images/block_map_storage.webp)

即使它需要大量内存, 我们仍然必须在某些时候存储体素数据. 这对于传递体素、存储无法使用生成器再现的玩家改动或将地形保存到磁盘非常重要.

做这件事的核心类是体素缓冲区[VoxelBuffer](api/VoxelBuffer.md).这是一个具有可配置格式的简单网格数据结构. 您可能不必经常使用此类, 但了解其存储概念对于使用其他 API（例如[VoxelTool](api/VoxelTool.md)很有用. 体素可以保存各种类型的值, 因此该对象使用多个通道, 每个通道都是一种类型的信息:

- `TYPE`: 体素的类型, 主要用于将体素与 Minecraft 中的模型类型相关联. 这与瓦片地图或网格地图中看到的瓦片 ID 非常相似.
- `SDF`: 有符号距离场值, 用于平滑体素. 它告诉体素距表面有多远. 这可以理解为物质的密度, 其中“-1”的值表示“物质”, “1”的值表示“空气”.
- `COLOR`: 颜色信息. 它可以是压缩的RGBA, 也可以是调色板的索引.
- 其他.

引擎使用第一个通道, 而其他通道暂时未使用. 每个通道还可以使用可配置的位深度: 8, 16, 32 或 64 bits. 这允许根据您的需要调整质量和内存使用情况.
最后, 该类做了一个小的内存优化，当通道内填充相同值时, 不会发生额外的内存分配. 


将体素保存到硬盘
-----------------------

![Raw region file seen as an image](images/region_file_seen_as_image.png)

当玩家对世界做出修改或者当你想要在编辑器内手动雕刻一个地形并保存时, 将体素保存到硬盘就是必要的操作.

该模块通过体素流[VoxelStream](api/VoxelStream.md)实现此功能. 和生成器类似, 一个流允许请求体素块，因此不需要一次性加载所有物体. 另外, 它也允许将一系列体素块输入保存.
如果世界很大，这种基于块的方法将特别有用. 相反, 较小的体积也许只需要一次性加载. 子类可以用多种方法来实现它, 通常使用压缩文件. 


将体素转化为网格体
----------------------------

![Cubes and wireframe](images/cubes_and_wireframe.webp)

现在的显卡性能越来越强了, 但是平均而言, 多边形(网格体)仍然是最快的3D模型渲染方法. 因此, 我们并不是真的直接去绘制体素. 作为替代, 我们需要先将他们转化成多边形, 然后再进一步渲染. 该引擎使用[VoxelMesher](api/VoxelMesher.md)来实现这一转化步骤.

有多种方法从体素数据转生成多边形数据, 该引擎提供了数种mesher转化器:

- [VoxelMesherCubes](api/VoxelMesherCubes.md):  体素的颜色被用来创建着色的方块. 这是最简单的体素多边形化方法. 
- [VoxelMesherBlocky](api/VoxelMesherBlocky.md): 体素的类型被用于将同类型网格体划分到同一批次进行处理. 这也可以使用方块, 但是提供自定义网格后也可以使用其他的形状. 这个方法和我的世界中的方法类似, 并且有很多设置选项. 
- [VoxelMesherTransvoxel](api/VoxelMesherTransvoxel.md): 该方法不使用方块, 而是使用SDF值, 基于[Transvoxel](https://transvoxel.org/)算法来创建平滑表面. 它还可以生成过渡网格体, 在实现两种不同网格体的缝合上非常有用. 
- [VoxelMesherDMC](api/VoxelMesherDMC.md): 使用双行进立方体生成光滑表面的变体, [介绍文章见链接](https://www.volume-gfx.com/volume-rendering/). 但是, 算法实现目前已经不维护了. [DMC](https://zhuanlan.zhihu.com/p/108459593?from_voters_page=true)


通过节点将体素放在一起
-------------------------------

![Game screenshots](images/game_examples.webp)

目前, 我们可以认为已经有足够的工具来在游戏中使用体素. 但是, 在此之前我们还有一些额外工作要做. 

对于单个模型或者一小块地形, 将一堆体素转化为单个网格体是可行的, 但是这种做法并不能很好地扩展到大片可编辑地形的情形. 如果地形需要有一个大的可视范围, 这个问题将更加棘手. 通常, 采取的解决方法是将其划分为区块, 最终使用多个细节层级, 并且在地形被玩家修改时正确更新被改动的部分. 这就是体素地形`VoxelTerrain*`节点的功能, 通过将前文所述的工具整合在一起, 并且使用多线程在后台执行繁重操作实现.

> 在这个引擎中, 区块"Chunks"被称为"Blocks". 它们通常代表 16x16x16 体素的立方体. 一些操作就是针对区块的, 而不是一般的空间单位. 

该引擎主要支持两类主要地形: 

- [VoxelTerrain](api/VoxelTerrain.md): 这个地形使用一个简单的区块网格, 当观察者移动时负责区块的加载和卸载. 它的视距和MC中的限制是类似的, 因此最好和盒状体素或者中等大小的平滑体积一起使用. 

- [VoxelLodTerrain](api/VoxelLodTerrain.md): 使用八叉树区块, 和简单网格不同, 这允许将体素以不同细节层级存储, 从而允许渲染相比而言远得多的距离. 但是, 它仅支持平滑体素, 并且使用上和前者有不同的限制. 

两种地形都支持实时编辑, 而且只有被编辑的部分会被动态重建. A helper class [VoxelTool](api/VoxelTool.md) is exposed to the script API to simplify the process of editing voxels, which can be obtained by using `get_voxel_tool()` methods. It allows to set single voxels, dig around or blend simple shapes such as spheres or boxes. The same concept of channels seen earlier is available on this API, so depending on the type of mesher you are using, you may want to edit either `TYPE`, `SDF` or `COLOR` channels.

Finally, for these terrains to load, it is required to place at least one [VoxelViewer](api/VoxelViewer.md) node in the world. These may be placed typically as child of the player, or the main `Camera3D`. They tell the voxel engine where to load voxels, how far, and give priority to mesh updates happening near them.

