<!-- markdownlint-disable -->
<p align="center">
  <img width="240" src="logo.png" style="text-align: center;"/>
</p>
<h1 align="center">comfyui-nodes-docs</h1>
<h4 align="center">Docs for every ComfyUI's node ✨ / ComfyUI节点文档✨</h4>

![Author: Mr.Huang & Leo](https://img.shields.io/badge/作者-Mr.Huang&Leo-blue.svg?style=for-the-badge)
[![ComfyuUI](https://img.shields.io/badge/ComfyUI-blue.svg?style=for-the-badge)](https://github.com/comfyanonymous/ComfyUI)

<!-- markdownlint-restore -->

中文文档｜ [English Document](README_en.md)

这是一个用于显示每个comfyui节点文档的插件。


![example1](examples/2.png)

## 安装

### comfyUI Manager

在comfyUI管理器中搜索`comfyui-nodes-docs`并安装。

### 自定义安装

- 在ComfyUI的插件目录中打开cmd窗口，例如"ComfyUI\custom_nodes"，输入`git clone https://github.com/CavinHuang/comfyui-nodes-docs` 或下载zip文件并解压，将生成的文件夹复制到ComfyUI\custom_ Nodes\

- 重新启动ComfyUI

## [节点列表](nodesList.md)

## 参与共建

### 两个方面：

- 参与插件的维护，修复问题，提升使用体验，优化代码

- 参与节点文档的建设，新增还未收录的节点文档，修改已有节点文档中不正确的地方，或者因为插件升级导致的文档滞后问题。

### 参与方式如下：

- Fork一份代码到你的github中

- 创建一个新的分支用于修改你的变化，在你的仓库中完成你所有的变化，并且提交。

- 创建一个Pull Request，提交你的变化分支合并申请到main分支

- 通过审核后，将会发布你的代码到最新的main分支，公众将可以使用你提交的特性。

### 添加新的节点文档

- 在`docs`文件夹中创建一个以`节点类型`命名的Markdown文件，例如`CLIPMergeSimple.md`

- 在文件中添加以下结构，请参考具体示例[CLIPMergeSimple.md](docs/CLIPMergeSimple.md)：

<pre><code>
# Documentation
- Class name: Node name
- Category: Node category
- Output node: False
- Repo Ref: https://github.com/xxxx

Description of nodes

# Input types

Node input types

# Output types

Node output types

# Usage tips
- Infra type: GPU

# Source code

Node source code
</code></pre>

## 更新日志

### 2024-05-26
- 修复了文档中的一些错误
- 更新了很多节点文档，可查看文档目录

### 2024-05-25
- 在设置中增加了开关，可以选择是否显示节点的文档
- 增加文档本地修改功能，如果觉得文档有问题，可以在本地修改，不会影响到其他人
- 在设置中增加了是否参与共建的开关，可以选择是否参与共建，默认打开，打开后会把本地修改的记录，记录到云DB上，后期经过审核后会合并到主分支上
- 增加导出文档和导入文档功能，导出文档会把本地修改的记录和仓库提供的文档导出下载，导入文档会把导出的文档导入到本地，不会影响主仓库的文档。
- 修复了一些bug