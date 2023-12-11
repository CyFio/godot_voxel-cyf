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


### Updating Your Build

If you cloned Godot and Voxel Tools, you can use git to update your local code.

1. Go to your local Godot source directory `godot` and run `git pull`. It will download all updates from the repository and merge them into your local source.
1. Go to `godot/modules/voxel` and run `git pull`. Git will update Voxel Tools.
1. Rebuild Godot.

!!! note
	Since you are pulling from two projects developped by different people, it's probable that on occasion your build won't compile, your project won't open, or your Voxel Tools won't work properly or even crash Godot. To minimize downtime, save your successful builds. Move them out of the build folder and rename them with the version number (e.g. godot-3.2+ee5ba3e.exe). This way, you can continue to use previously working builds until the Godot or Voxel developers fix whatever is broken. It is generally desired by all that code published to repositories will at least build, but stuff happens.


C# suppport
--------------

### Module

C# builds are available on Github Actions as well (as "Mono Builds"). Unfortunately, Godot 4 changed the way C# integrates by using the Nuget package manager. This made it really inconvenient for module developers to provide ready-to-use executables, and hard for users too:

- When you make a project in Godot C#, it fetches the "vanilla" Godot SDK from Nuget, but it is only available for official stable versions, so you can't use CI builds of the engine that use the latest development version of Godot.
- Modules add new classes to the API which are not present in the official SDK. It would require to create SDKs for every combination of modules you want to use and upload them to Nuget, which isn't practical.
- You could revert to the latest official SDK available on Nuget, but to access module APIs you would have to use workarounds such as `obj.Get(string)`, `Set(string)` and `Call(string, args)` in code, which is hard to use, inefficient and terrible to maintain.

To obtain a working version, you may generate the SDK yourself and use a local Nuget repository instead of the official one. Follow the steps described in the [Godot Documentation for C#](https://docs.godotengine.org/en/stable/contributing/development/compiling/compiling_with_dotnet.html).


### GDExtension and C#

The module can also compile as a GDExtension library, which doesn't require to build Godot. However, C# support of extensions implemented in C++ is not well defined at the moment.


Export templates
-------------------

### Getting a template

In Godot Engine, exporting your game as an executable for a target platform requires a "template". A template is an optimized build of Godot Engine without the editor stuff. Godot combines your project files with that template and makes the final executable.

If you only download the Godot Editor with the module, it will allow you to develop and test your game, but if you export without any other setup, Godot will attempt to use a vanilla template, which won't have the module. Therefore, it will fail to open some scenes.

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
