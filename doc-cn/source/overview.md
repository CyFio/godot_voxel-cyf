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

The engine uses the first channels, while others are unused for now. Each channel can also use a configurable bit depth: 8, 16, 32 or 64 bits. This allows to tune quality and memory usage depending on your needs.
Finally, a simple optimization is applied so that if a channel is filled with the same value, it won't allocate memory and instead store just that value. This way, areas such as the sky don't take up memory.


Saving voxels to disk
-----------------------

![Raw region file seen as an image](images/region_file_seen_as_image.png)

When players make edits to the world or when you want to sculpt a terrain in the editor and keep your changes saved, it becomes necessary to save voxels to disk.

This module implements this functionality with [VoxelStream](api/VoxelStream.md). Similarly to generators, a stream allows to request blocks of voxels so that it's not necessary to load the entire thing at once. But also, it allows to send back blocks of voxels to save them.
This block-based approach is especially useful if the world is very large. In contrast, smaller volumes might just be loaded all at once. Subclasses may implement it in various ways, often using compressed files.


Turning voxels into meshes
----------------------------

![Cubes and wireframe](images/cubes_and_wireframe.webp)

Today's graphics cards are getting more and more powerful, but in average, polygons (meshes) remain the fastest way to render 3D models. So we are not really going to directly draw voxels. Instead, we have to convert them into polygons, which will then be rendered. To do this, this engine uses a resource called [VoxelMesher](api/VoxelMesher.md).

There are several ways to produce polygons from voxel data, and the engine provides types of meshers to do it:

- [VoxelMesherCubes](api/VoxelMesherCubes.md): the color of voxels is used to produce colored cubes. This is the simplest way to polygonize voxels.
- [VoxelMesherBlocky](api/VoxelMesherBlocky.md): the type of voxels is used to batch together meshes corresponding to that type. This can also use cubes, but any shape will do if custom meshes are provided. This is a similar technique as the one used in Minecraft, and has a wide range of options.
- [VoxelMesherTransvoxel](api/VoxelMesherTransvoxel.md): instead of making cubes, this one uses the SDF value to produce a smooth surface, based on the [Transvoxel](https://transvoxel.org/) algorithm. It can also produce transition meshes, which is useful to stitch together two meshes of different level of detail.
- [VoxelMesherDMC](api/VoxelMesherDMC.md): variant using dual marching cubes to produce a smooth surface, [as explained in these articles](https://www.volume-gfx.com/volume-rendering/). However this implementation is no longer maintained.


Putting it together with nodes
-------------------------------

![Game screenshots](images/game_examples.webp)

So far, we could consider that there are enough tools to use voxels within games. It's a bit more work to take it from there, though.

Turning a bunch of voxels into a mesh is ok for a model or a small piece of land, however it won't scale well with large editable terrains. It gets more tricky if that terrain needs to have a large view distance. Very often, the proposed solution is to split it into chunks, eventually using variable level of detail, and properly update parts of that terrain when they are modified by players. This is what the `VoxelTerrain*` nodes do, by putting together the tools seen before, and using threads to run heavy operations in the background.

!!! note
    In this engine, "Chunks" are actually called "Blocks". They typically represent cubes of 16x16x16 voxels. Some options are specified in blocks, rather than spatial units.

There are two main types of terrains supported by the engine:

- [VoxelTerrain](api/VoxelTerrain.md): this one uses a simple grid of blocks, and takes care of loading and unloading blocks as the viewer moves around. Its view distance is limited in similar ways to Minecraft, so it is better used with blocky voxels or moderate-size smooth volumes.

- [VoxelLodTerrain](api/VoxelLodTerrain.md): this one uses an octree of blocks. Contrary to a simple grid, this allows to store voxels at multiple levels of detail, allowing to render much larger distances. It only supports smooth voxels though, and has different limitations compared to the other.

Both kinds of terrains can be edited in real time, and only the edited parts will be re-meshed dynamically. A helper class [VoxelTool](api/VoxelTool.md) is exposed to the script API to simplify the process of editing voxels, which can be obtained by using `get_voxel_tool()` methods. It allows to set single voxels, dig around or blend simple shapes such as spheres or boxes. The same concept of channels seen earlier is available on this API, so depending on the type of mesher you are using, you may want to edit either `TYPE`, `SDF` or `COLOR` channels.

Finally, for these terrains to load, it is required to place at least one [VoxelViewer](api/VoxelViewer.md) node in the world. These may be placed typically as child of the player, or the main `Camera3D`. They tell the voxel engine where to load voxels, how far, and give priority to mesh updates happening near them.
