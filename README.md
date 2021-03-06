---
layout: page
---
# 学习笔记

## 编写说明

### 语言
- [LyX](./documenting/latex/README.md#LyX)
  - 《[数学](#数学)》《[物理](#物理)》《[算法](#算法)》含有大量数学公式，因此整理为 LyX 文档。
  - 本页内的链接指向可单独编译的分卷，不同分卷可能含有重复的章节。
  - [顶层 `README.lyx`](./README.lyx) 大致按逻辑顺序重新编排了章节（重复的只保留一份）。
- [Markdown](./programming/markdown.md)
  - 《[编程](#编程)》《[文档](#文档)》含有大量代码，因此整理为 Markdown 文档。
  - 本仓库已启用 [**GitHub Pages**](https://pvcstillingradschool.github.io/miniWiki/)，目前仅支持 Markdown 文档。

### 编译

- 在任意路径下创建本仓库的副本。
- 编译 LyX 文档（生成 PDF 文件）：
  1. 安装 [TeX Live](./documenting/latex/README.md#TeX-Live) 和 [LyX](./documenting/latex/README.md#LyX)，并开启 [中文支持](./documenting/latex/README.md#中文支持) 及 [代码高亮](./documenting/latex/README.md#代码高亮)。
  1. 安装 [STIX](https://github.com/stipub/stixfonts) 字体。
  1. 如果要编译除 [顶层 `README.lyx`](./README.lyx) 以外的其他 LyX 或 TeX 文档，则需在本地 `$TEXMFHOME/tex/latex` 下创建一个指向 [`miniWiki/documenting/latex/pvcstyle.sty`](./documenting/latex/pvcstyle.sty) 的 ***符号链接 (symbolic link)***：
    
     |  操作系统  | `TEXMFHOME` 的默认值 |  创建符号链接的命令  |
     | :--------: | :------------------: | :------------------: |
     |   macOS    |  `~/Library/texmf`   | `ln -s TARGET LINK`  |
     | Windows 10 |      `~/texmf`       | `mklink LINK TARGET` |

- 编译 Markdown 文档（生成 HTML 文件）：
  1. 打开命令行终端，安装 [prerequisites](https://help.github.com/en/github/working-with-github-pages/testing-your-github-pages-site-locally-with-jekyll#prerequisites)。
  1. 进入本仓库根目录，在命令行中输入
     ```shell
     $ bundle exec jekyll serve
     ```
  1. 生成的文件位于 `./_site/` 中。
  1. 在浏览器中访问 [`http://127.0.0.1:4000/`](http://127.0.0.1:4000/) 或 [`http://localhost:4000/`](http://localhost:4000/)。

### 排版

- 【空格】以下情形手动空一格：
  - 英文单词、行内公式、行内代码 两侧（左侧为行首、右侧为标点者除外）
  - 可能因断词产生歧义的汉字之间（例如「以 光武帝之德」与「以光 武帝之德」）
- 【待补充】

### 纠错
移步 [Issues](https://github.com/pvcStillInGradSchool/miniWiki/issues)，尽量做到一个 issue 对应一个问题。

## 内容目录

### [数学](./mathematics/README.md)

|                       卷                        |  章  |  节  |  进度   |
| :---------------------------------------------: | :--: | :--: | :-----: |
|  [线性代数](./mathematics/algebra/README.lyx)   |      |      | `00001` |
|     [实分析](./mathematics/real/README.lyx)     |      |      | `00001` |
|   [复分析](./mathematics/complex/README.lyx)    |      |      | `11111` |
| [泛函分析](./mathematics/functional/README.lyx) |      |      | `00011` |
|   [常微分方程](./mathematics/ode/README.lyx)    |      |      | `00111` |
|   [偏微分方程](./mathematics/pde/README.lyx)    |      |      | `00111` |
|  [微分几何](./mathematics/geometry/README.lyx)  |      |      | `00001` |

### [物理](./physics/README.md)

|                    卷                    |  章  |  节  |  进度   |
| :--------------------------------------: | :--: | :--: | :-----: |
|   [热力学](./physics/heat/README.lyx)    |      |      | `00001` |
|  [流体力学](./physics/fluid/README.lyx)  |      |      | `00111` |
| [量子力学](./physics/quantum/README.lyx) |      |      | `00011` |

### [算法](./algorithms/README.md)

|                           卷                           |  章  |  节  |  进度   |
| :----------------------------------------------------: | :--: | :--: | :-----: |
| [数值分析](./algorithms/numerical_analysis/README.lyx) |      |      | `00001` |
|      [优化](./algorithms/optimization/README.lyx)      |      |      | `00001` |
|    [有限元](./algorithms/finite_element/README.lyx)    |      |      | `00011` |

### [编程](./programming/README.md)
- 计算机系统
  - [CSAPP](./programming/csapp/README.md)
- 设计思想
  - [设计原则](./programming/principles/README.md)
  - [设计模式](./programming/patterns/README.md)
- 编程语言
  - [C++](./programming/cpp/README.md)
  - [Octave/MATLAB](./programming/octave.md)
  - [Python](./programming/python.md)
  - [UML](./programming/uml/README.md)
    - [PlantUML](./programming/uml/README.md#PlantUML)
- 并行计算
  - [MPI](./programming/mpi/README.md)
- 网格离散
  - [CGNS](./programming/cgns/README.md)
  - [Gmsh](./programming/gmsh/README.md)
  - [VTK](./programming/vtk/README.md)
    - [ParaView](./programming/vtk/README.md#ParaView)
- 构建工具
  - [版本控制](./programming/git.md)
    - [Git](./programming/git.md#Git)
    - [GitHub](./programming/git.md#GitHub)
  - [批量构建](./programming/make/README.md)
    - [GNU Make](./programming/make/README.md#GNU-Make)
    - [CMake](./programming/make/README.md#CMake)
  - [断点调试](./programming/debug/README.md)
    - [GDB](./programming/debug/README.md#GDB)
    - [LLDB](./programming/debug/README.md#LLDB)
- 开发环境
  - [Linux](./programming/linux/README.md)
    - [安装与配置](./programming/linux/install/README.md)
    - [SSH](./programming/linux/ssh.md)
    - [Vim](./programming/linux/vim.md)
  - [Docker](./programming/docker/README.md)

### [文档](./documenting/README.md)
- [LaTeX](./documenting/latex/README.md)
  -  [LyX](./documenting/latex/README.md#LyX)
- [Markdown](./documenting/markdown.md)
  -  [Typora](./documenting/markdown.md#Typora)
- [编辑书签](./documenting/bookmark)
  - [PDFBookmarker](./documenting/bookmark.md#PDFBookmarker)
  - [DJVUSED](./documenting/bookmark.md#DJVUSED)
