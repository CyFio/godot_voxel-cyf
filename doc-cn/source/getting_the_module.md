获取体素插件
=====================

这是一个C++模块，需要编译进Godot引擎

已经编译好的版本
-------------------

### 发布版

目前没有对每个里程碑构建，我们只对最新版本构建开发版。

因为插件是作为一个模块开发的，因此需要对Godot编辑器做一个完整的自定义编译，这也许会和官方版本不同。

引擎本身比较大并且目标平台多，但我们的模块相比要小且我们没有专属的构建容器，因此不是所有的导出模板都可以获取。你可以通过开发你的游戏并且使用主要的桌面平台测试，但是如果你想导出它，你也许需要自行编译这些模板。

### 开发版

#### 面向Godot 4

在Github->Actions中可获取，选择你的平台：

- [Windows构建](https://github.com/Zylann/godot_voxel/actions/workflows/windows.yml)
- [Linux构建](https://github.com/Zylann/godot_voxel/actions/workflows/linux.yml)
- [Mono构建](https://github.com/Zylann/godot_voxel/actions/workflows/windows.yml) (很可能不可用，它们在Godot 4以后就寄了，需要修复帮助)

然后，单击勾选最新的成功构建，出现绿色勾

![Screenshot of a list of builds, with the latest successful one circled in green](images/ci_builds_latest_link.webp)

再滑动至底部，你应该会看到下载链接：

![Github actions screenshot](images/github_actions_windows_artifacts.webp)

如果存在多个可下载项目，编辑器的构建将是名字带有`editor`的那个。

这些版本对应[changelog](https://github.com/Zylann/godot_voxel/blob/master/CHANGELOG.md)中描述的`master`版本。
它们使用Godot最新的稳定版本分支构建(比如，本文撰写的时间是`4.0`)，而不是Godot的`master`分支，除非特别说明。
每次提交推送到主分支以及其他开发者创建拉去请求时都会有一次全新的构建，因此您在选择构建版本时需要仔细考虑。


!!! 注意
	你需要一个Github账号来下载开发版本，否则，链接将无法工作。

#### 面向Godot 3(遗留版本)

在Github action中，你也许需要查找`godot3.x`分支下的版本。

但是，在这些版本上的新特性开发已经停止了，这意味着下载已过期时可能的。在这种情况下，你也许需要找一个发布版本或者自行编译。

### Tokisan 构建 (时先生?非常老)

- 点击链接获取[Tokisan Games website](http://tokisan.com/godot-binaries/).

很久以前， [Cory Petkovsek](https://github.com/tinmanjuggernaut)完整构建Godot并自行添加了一些附加模块，包括导出模板。但是，它们现在可能已经落后于当前版本，因此丢失了许多近期特性和设置差异。

自行编译
-------------------

这些步骤能帮助你创建一个包含体素插件的自定义Godot编译版本。

### 构建Godot

1. 根据 [官方指南](https://docs.godotengine.org/en/latest/development/compiling/index.html)下载并编译 [Godot 源码](https://github.com/godotengine/godot) . 如果你需要经常更新你的构建 (推荐), 请使用Git克隆该仓库而不是下载zip文件。
2. 请确保选择了合适的分支。如果你想要最新的开发版本，请使用Godot的`master`分支。如果你想要构建一个的更加稳定的发布版本，使用那个版本对应的分支 (比如`4.0`)或者一个特定的版本标签(比如 `4.0.2-stable`)。如果你想用Godot 3，请使用Godot的`3.x`分支，和模块的`godot3.x`分支 (但是这已经不再维护了)。 
3. 在添加模块之前，请先构建Godot并且确保构建成功，这会生成可执行文件。
4. 运行`godot/bin`下新构建的可执行文件。请查看帮助&关于菜单以确认版本号是否和您需要的一致(例如 `3.2dev.custom_build.ee5ba3e`)。


### 添加体素插件

1. 下载或克隆[体素工具](https://github.com/Zylann/godot_voxel)仓库。如果你想更轻松地更新你的构建，请使用Git克隆(推荐)。
2. 默认情况，模块的`master`分支可以在Godot的最新稳定分支下工作。模块有一些“快照”版本，它们在特定的Godot发布版本时创建，但之后不会更新(比如 `godot4.0`)。
3. 将体素工具文件夹放入你的Godot源码树下的 `godot/modules` 目录。 
4. 将体素工具文件夹重命名为`voxel`。如果正确，文件将被定位到(比如 README.md)`godot/modules/voxel`目录下。 **这一步很重要！**
5. 重新构建Godot并确保能输出可执行文件。
6. 测试你的构建版本是否提供体素支持：
	1. 运行你的新Godot版本。
	2. 创建一个新项目。
	3. 创建一个新3D场景。
	4. 添加一个新节点，查找"Voxel"并检查是否出现 "VoxelTerrain"。如果出现了，您的编译成功了。否则，请重新检查之前的步骤，查看是否有错过了某一步骤或者中途某些过程失败了。


### 更新你的构建版本

如果您克隆了Godot和体素插件，你可以使用git来更新你本地的代码。

1. 来到您本地的Godot源文件路径`godot`并且运行`git pull`。这样会从仓库拉去所有更新并且将他们合并到您的本地源码中。
2. 来到`godot/modules/voxel`并且运行`git pull`。Git将更新体素插件。
3. 重新构建Godot

!!! note
	因为你从两个不同人开发的项目进行拉取，所以可能会构建失败，导致你的项目不能打开或者你的体素插件无法正确工作甚至使Godot崩溃。为了减小时间上的损失，请留意保存成功构建的版本，将它们从build文件夹中移出并使用版本号重命名，(例如godot-3.2+ee5ba3e.exe)。通过这种方式，你仍然能过继续使用之前的版本，直到开发者修复问题。虽然通常大家都希望所有发布的代码至少能过通过编译，但避免不了出现错误的情况。

C#支持
--------------

### 模块

C#版本在Github Actions中也可获取(作为 "Mono 版本")。 不巧的是，由于Godot4将C#的集成方式改为使用Nuget包管理工具，这使得模块开发者提供开箱即用的可执行文件夹非常不便，而且使使用者会遇到以下困难：

- 当你使用C#制作Godot项目时，它会使用Nuget获取"原版"(vanilla)的SDK，但它只支持官方稳定版本，所以你无法使用最新的引擎开发版本进行持续集成(CI)构建。
- 模块中含有官方SDK没有包含的新类型和API。这要求为您想要使用的每个模块组合创建 SDK 并将它们上传到 Nuget，这是不切实际的.
- 你可以回退到Nuget上最新的官方SDK，但是这要求您使用`obj.Get(string)`, `Set(string)`，`obj.Get(string)`, `Set(string)`和`Call(string, args)`这些难以使用、低效且难以维护的代码来访问模块的API。

要获得工作版本，您可以自己生成 SDK 并使用本地 Nuget 存储库而不是官方存储库。 按照 [Godot C#文档](https://docs.godotengine.org/en/stable/contributing/development/compiling/compiling_with_dotnet.html). 中描述的步骤操作


### GDExtension和C#

该模块还可以编译为 GDExtension 库，不需要构建 Godot。 然而，C# 对 C++ 中实现的扩展的支持目前尚未明确定义。

导出模板
-------------------

### 获取一个模板

在Godot引擎中，将你的游戏导出为目标平台的可执行文件需要一个“模板”。模板是Godot引擎移除编辑器相关部分的优化版本。Godot将你的项目文件和该模板结合生成最终的可执行文件。

如果你仅下载了包含该模块的Godot编辑器，它将允许你开发和测试你的游戏，但时如果你不进行其他设置就导出，Godot将尝试使用原版的模板，这个模板不含有该模块。因此，它将无法打开某些场景。

As mentionned in earlier sections, there are currently no "official" builds of this module, but you can get template builds at the same place as [latest development versions](#development-builds). Template builds are those with `template` in their name.

If there is no template available for your platform, you may build it yourself. This is the same as building Godot with the module, only with different options. See the [Godot Documentation](https://docs.godotengine.org/en/latest/development/compiling/index.html) for more details, under the "building export templates" category of the platform you target.

### Using a template

Once you have a template build, tell Godot to use it in the Export configurations. Fill in the path to a custom template in the "Custom Template" section:

![Screenshot of Godot export configuration window with a custom template assigned for Windows](images/export_template_window.webp)

### GDExtension

In the future, it is hoped that none of this manual work is required. Rather than getting a custom version of the engine, you would download Voxel from the asset library. Developping, testing, exporting would just work without extra setup.

GDExtension is what would make this possible. This module can already [compile with GodotCpp](module_development.md#gdextension) so it can be loaded as a GDExtension library.
Unfortunately, there are still [too many problems](https://github.com/Zylann/godot_voxel/issues/442) for the module to work properly. For the time being, a custom engine build is more reliable.

There are development builds available on [Github Actions](https://github.com/Zylann/godot_voxel/actions/workflows/extension_windows.yml) as we try to keep the module compiling with GodotCpp, however they lack testing and might crash. Use at your own risk.
