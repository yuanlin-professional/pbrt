pbrt，第 4 版（早期发布）
===============================

[<img src="https://github.com/mmp/pbrt-v4/workflows/cpu-linux-build-and-test/badge.svg">](https://github.com/mmp/pbrt-v4/actions?query=workflow%3Acpu-linux-build-and-test)
[<img src="https://github.com/mmp/pbrt-v4/workflows/cpu-macos-build-and-test/badge.svg">](https://github.com/mmp/pbrt-v4/actions?query=workflow%3Acpu-macos-build-and-test)
[<img src="https://github.com/mmp/pbrt-v4/workflows/cpu-windows-build-and-test/badge.svg">](https://github.com/mmp/pbrt-v4/actions?query=workflow%3Acpu-windows-build-and-test)
[<img src="https://github.com/mmp/pbrt-v4/workflows/gpu-build-only/badge.svg">](https://github.com/mmp/pbrt-v4/actions?query=workflow%3Agpu-build-only)

![Transparent Machines 帧，来自 @beeple](images/teaser-transparent-machines.png)

这是 pbrt-v4 的早期发布版本，该渲染系统将在即将出版的第四版《*基于物理的渲染：从理论到实现*》中进行描述。（印刷版将于 2023 年 2 月中旬发售；部分章节将于 2022 年秋末提前公开；完整内容将在出版后六个月免费提供，与[第三版](https://pbr-book.org)一样。）

我们将此代码提供给勇于探索的开发者；虽然文档尚未完善，但如果你熟悉 pbrt 的早期版本，应该能够顺利上手。我们希望该系统在当前形式下对部分开发者有所帮助，同时也希望在书籍最终定稿之前发现并修正当前实现中的 bug。

资源
---------

* pbrt-v4 的多个场景文件可在 [git 仓库](https://github.com/mmp/pbrt-v4-scenes)中获取。
* [pbrt-v4 用户指南](https://pbrt.org/users-guide-v4.html)。
* [pbrt-v4 场景描述格式](https://pbrt.org/fileformat-v4.html)文档。

功能特性
--------

pbrt-v4 相较于之前的 pbrt-v3 进行了大幅更新。主要变化包括：

* 光谱渲染
  * 渲染计算始终使用点采样光谱进行；RGB 颜色仅用于场景描述（如图像纹理贴图）和最终图像输出。
* 现代化的体积散射
  * 新增了基于 [Miller 等人 2019](https://cs.dartmouth.edu/~wjarosz/publications/miller19null.html) 的零散射路径积分公式的全新 `VolPathIntegrator`。
  * 通过独立的低分辨率 majorant 网格，`GridDensityMedium` 使用了更紧凑的零散射 majorant。
  * 现在支持发光体积以及具有 RGB 值吸收和散射系数的体积。
* 在配备 CUDA 和 OptiX 的系统上支持 GPU 渲染。
  * GPU 路径提供了基于 CPU 的 `VolPathIntegrator` 的全部功能，包括体积散射、次表面散射、pbrt 的所有相机、采样器、形状、光源、材质和 BxDF 等。
  * 性能远超 CPU 渲染。
* 新的 BxDF 和材质
  * 提供的 BxDF 和材质已重新设计，与物理散射过程更紧密关联，类似于 Mitsuba 的材质系统。（除此之外，功能大而全的 UberMaterial 已被移除。）
  * 测量 BRDF 现在使用 [Dupuy 和 Jakob 的方法](https://rgl.epfl.ch/publications/Dupuy2018Adaptive)表示。
  * 分层材质的散射使用蒙特卡洛随机游走进行精确模拟（基于 [Guo 等人 2018](https://shuangz.com/projects/layered-sa18/)）。
* 实现了多种光源采样改进。
  * 通过光源 BVH 实现了"多光源"采样（[Conty 和 Kulla 2018](http://aconty.com/pdf/many-lights-hpg2018.pdf)）。
  * 三角形（[Arvo1995](https://dl.acm.org/doi/10.1145/218380.218500)）和四边形（[Ureña 等人 2013](https://www.arnoldrenderer.com/research/egsr2013_spherical_rectangle.pdf)）光源使用立体角采样。
  * 间接光照和 BSDF 采样的直接光照现在只需追踪一条光线。
  * 使用 warp product 采样进行近似余弦加权立体角采样（[Hart 等人 2019](https://onlinelibrary.wiley.com/doi/abs/10.1111/cgf.14060)）。
  * 包含了 Bitterli 等人的环境光[传送门采样](https://benedikt-bitterli.me/pmems.html)技术的实现。
* 渲染现在可以使用绝对物理单位，并根据 [Langlands 和 Fascione 2020](https://github.com/wetadigital/physlight) 对真实相机进行建模。
* 其他改进...
  * 对 `Sampler` 类进行了多项改进，包括更好的随机化以及实现了 [Ahmed 和 Wonka 的蓝噪声 Sobol 采样器](http://abdallagafar.com/publications/zsampler/)的新采样器。
  * 新增了 `GBufferFilm`，可提供每个像素的位置、法线、反照率等信息。（这对降噪和机器学习训练特别有用。）
  * 路径正则化（可选）。
  * 新增了双线性面片图元（[Reshetov 2019](https://link.springer.com/chapter/10.1007/978-1-4842-4427-2_8)）。
  * 多项光线-形状相交精度改进。
  * 大部分底层采样代码已被提取为独立函数，便于复用。同时提供了许多采样技术的逆函数。
  * 单元测试覆盖率大幅提升。

我们还对整个系统进行了重构，清理了各种 API 和数据类型，以提高可读性和易用性。

最后，pbrt-v4 可以与 [tev](https://github.com/Tom94/tev) 图像查看器配合使用，在渲染过程中实时显示图像。在最新版本中，*tev* 可以通过网络套接字接收图像；默认监听端口 14158，可通过 ``--hostname`` 命令行选项更改。如果你正在运行 *tev* 实例，可以这样运行 pbrt：
```bash
$ pbrt --display-server localhost:14158 scene.pbrt
```
这样，图像将在渲染过程中逐步显示。

构建代码
-----------------

与之前一样，pbrt 使用 git 子模块来管理多个所依赖的第三方库。因此，克隆仓库时请务必使用 `--recursive` 标志：
```bash
$ git clone --recursive https://github.com/mmp/pbrt-v4.git
```

如果你不小心在克隆 pbrt 时没有使用 ``--recursive``（或者在添加了新的子模块后需要更新 pbrt 源码树），请运行以下命令来获取依赖项：
```bash
$ git submodule update --init --recursive
```

pbrt 使用 [cmake](http://www.cmake.org/) 作为构建系统。注意默认为 Release 构建；如需 Debug 构建，请向 cmake 传递 `-DCMAKE_BUILD_TYPE=Debug`。

pbrt 应该能在任何支持 C++17 的 C++ 编译器系统上构建；我们已验证它可以在 Ubuntu 20.04、MacOS 10.14 和 Windows 10 上构建。欢迎提交 PR 来修复阻止在其他系统上构建的问题。

Bug 报告和 PR
-------------------

请使用 [pbrt-v4 GitHub Issue 追踪器](https://github.com/mmp/pbrt-v4/issues)报告 pbrt-v4 中的 bug。（我们已经预先填写了一些与初始发布中已知 bug 对应的 issue。）

我们随时欢迎修复 bug 的 Pull Request，包括你自己发现的 bug 或 Issue 追踪器中已有问题的修复。我们也乐于听取关于我们所实现的各种算法的改进建议。

但请注意，为了在有限时间内完成本书，pbrt-v4 的功能基本上已经固定。因此，我们不会接受对系统运行方式或结构进行重大更改的 PR（但欢迎在你自己的 fork 中保留这些更改！）。此外，不要为源代码中标记为 "TODO" 或 "FIXME" 的内容提交 PR；我们会在最终打磨时处理这些问题。

升级 pbrt-v3 场景
-----------------------

输入文件格式有多项更改，如上所述，新格式尚未完全文档化。不过，pbrt-v4 通过提供自动升级机制进行了部分弥补：
```bash
$ pbrt --upgrade old.pbrt > new.pbrt
```

大多数场景文件可以自动升级。在某些情况下需要手动干预；此时会打印错误信息。

环境贴图参数化方式也发生了变化（从等距矩形映射改为等面积映射）；你可以使用以下命令升级环境贴图：
```bash
$ imgtool makeequiarea old.exr --outfile new.exr
```

将场景转换为 pbrt 文件格式
---------------------------------------

导入场景到 pbrt 的最佳选择是使用 [assimp](https://www.assimp.org/)，自 2021 年 1 月 21 日起，assimp 已支持导出为 pbrt-v4 文件格式：
```bash
$ assimp export scene.fbx scene.pbrt
```

虽然转换器会尝试将材质转换为 pbrt 的材质模型，但导出后可能需要进行一些手动调整。此外，面光源并不总能被成功检测到，可能也需要手动处理。建议在转换后使用 pbrt 内置的网格转换功能将其转为二进制 PLY 格式。（`pbrt --toply scene.pbrt > newscene.pbrt`）。

在 GPU 上使用 pbrt
---------------------

要在 GPU 上运行，pbrt 需要：

* GPU 上的 C++17 支持，包括使用 C++ lambda 表达式进行内核启动。
* 统一内存，以便 CPU 可以为 GPU 上运行的代码分配和初始化数据结构。
* GPU 上的光线-物体相交 API。

这些要求实际上使得 pbrt 能够在对核心系统进行有限修改的情况下移植到 GPU。在实践中，这些功能目前仅通过 NVIDIA GPU 上的 CUDA 和 OptiX 可用，但我们很乐意看到 pbrt 在提供这些功能的任何其他 GPU 上运行。

pbrt 的 GPU 路径目前需要 CUDA 11.0 或更高版本以及 OptiX 7.1 或更高版本。支持 Linux 和 Windows。

构建脚本会自动尝试在常见位置查找 CUDA 编译器；cmake 输出会指示是否找到成功。需要手动设置 cmake 的 `PBRT_OPTIX_PATH` 配置选项以指向 OptiX 安装目录。默认情况下，pbrt 目标 GPU 着色器模型会根据系统中的 GPU 自动设置。也可以手动设置 `PBRT_GPU_SHADER_MODEL` 选项（例如 `-DPBRT_GPU_SHADER_MODEL=sm_80`）。

即使编译时启用了 GPU 支持，pbrt 默认仍使用 CPU，除非指定 `--gpu` 命令行选项。注意使用 GPU 渲染时，`--spp` 命令行标志可以方便地增加每像素采样数。此外，使用 *tev* 观看渲染进度也很有趣。

作为 pbrt 构建的一部分，imgtool 程序在 GPU 构建中支持 OptiX 降噪器。降噪器可以处理仅含 RGB 的图像，但对于包含辅助通道（如反照率和法线）的"深度"图像效果更好。将场景的 "Film" 类型设置为 "gbuffer" 并使用 EXR 作为图像格式进行渲染，pbrt 就会生成这样的"深度"图像。无论哪种情况，使用降噪器都很简单：
```bash
$ imgtool denoise-optix noisy.exr --outfile denoised.exr
```
