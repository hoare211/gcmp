# gcmp - Git Compare on Steroids

`gcmp` (Git Compare) 是一个强大的命令行脚本，它利用 Beyond Compare（或其他图形化比较工具）来直观地比较 Git 仓库中的任意两个“区域”。它将 `git diff` 的灵活性与图形化工具的易用性完美结合，极大地提升了代码审查和变更分析的效率。

 <!-- 建议：如果你能录制一个简短的GIF演示，放在这里会非常吸引人 -->

---

## ✨ 核心功能 (Core Features)

*   **直观的关键字**: 使用简单的关键字 `work`, `stage`, `stash` 即可代表工作区、暂存区和最新的储藏。
*   **全区域比较**: 支持在 **工作区 (Working Directory)**、**暂存区 (Staging Area)**、**储藏 (Stash)** 以及任意 **commit、分支、标签** 之间进行比较。
*   **智能标签**: 自动为比较的源生成清晰的标签（如分支名或 `work`），并在比较工具的标题栏和临时目录中显示，一目了然。
*   **目录级比较**: 将 Git 的状态导出为完整的目录树进行比较，让你能从宏观上把握所有文件的变更。
*   **高度可定制**: 可以轻松将默认的 `bcompare` 更换为你喜欢的任何其他比较工具（如 `kdiff3`, `meld`, `vscode` 等）。
*   **安全可靠**: 所有操作均在临时目录中进行，脚本退出后会自动清理，绝不污染你的工作区。

## ❓ 为何使用 gcmp? (Why use `gcmp`?)

标准的 `git diff` 非常强大，但它在终端中逐文件显示 `diff` 结果，对于大型或分散的变更，难以形成整体认知。

`gcmp` 解决了以下痛点：

1.  **宏观视角**: 想知道 `feature-A` 分支相对于 `main` 分支到底改了哪些文件、增删了哪些文件？`gcmp` 能以目录树的形式清晰展示。
2.  **复杂比较简化**: `git diff work stage` 这样的操作不存在于原生 Git 中。而 `gcmp work stage` 却能轻松实现，让你清晰地看到工作区哪些修改尚未 `git add`。
3.  **图形化优势**: 对于复杂的单文件变更，图形化比较工具的左右同步滚动、高亮显示、行内差异等功能，远比终端中的 `+` 和 `-` 更直观。

## 🚀 安装 (Installation)

我们推荐将 `gcmp` 脚本放置在你的系统 `PATH` 中，以便在任何仓库中都能方便地调用。


### 手动安装

1.  下载 `gcmp`。
2.  在终端中为它添加执行权限: `chmod +x gcmp`。
3.  将此文件移动到一个包含在你 `PATH` 环境变量中的目录，例如 `~/bin` 或 `/usr/local/bin`。

## 💡 使用方法 (Usage)

### 基本语法

```
gcmp [repo_path] <ref1> <ref2>
```

*   `repo_path` (可选): Git 仓库的路径。如果省略，则使用当前目录。
*   `<ref1>` 和 `<ref2>`: 两个要比较的 Git 引用或关键字。

### 特殊关键字 (Special Keywords)

这是 `gcmp` 的精髓所在，让复杂的比较变得简单。

| 关键字 (Keyword) | 代表的 Git 区域 | 说明 |
| :--------------- | :------------------ | :----------------------------------------------------------- |
| `work`           | 工作区 (Working Dir) | 所有被 Git 跟踪的、在工作目录中的文件。 |
| `stage`          | 暂存区 (Staging Area) | 已经通过 `git add` 添加，准备下次提交的内容。 |
| `stash`          | 最新储藏 (Stash)   | 代表 `stash@{0}`，即你最近一次执行 `git stash` 保存的内容。 |

除了这些关键字，`<ref1>` 和 `<ref2>` 也可以是任何 Git 能识别的引用（commit-ish），例如：
*   **Commit HASH**: `abc123d`, `7a4e3f81`
*   **分支名 (Branch)**: `main`, `my-feature`
*   **标签名 (Tag)**: `v1.0.2`
*   **相对引用 (Relative Ref)**: `HEAD`, `HEAD~2`, `main~1`
*   **其他储藏 (Other Stashes)**: `'stash@{1}'` (注意使用引号)

### 常用示例 (Common Examples)

---

#### 1. 比较工作区与暂存区
*查看哪些修改还没有被 `git add`*
```sh
gcmp work stage
```
**它回答了**: "如果我现在执行 `git add .`，会有哪些变更被添加到暂存区？"

---

#### 2. 比较暂存区与上次提交
*预览你将要提交的内容*
```sh
gcmp stage HEAD
```
**它回答了**: "如果我现在执行 `git commit`，会产生一个什么样的快照？" (这等价于 `git diff --staged` 的图形化版本)

---

#### 3. 比较工作区与上次提交
*查看自上次提交以来的所有本地修改（包括已暂存和未暂存的）*
```sh
gcmp work HEAD
```
**它回答了**: "相比我上次提交，我到底改了些什么？" (这等价于 `git diff HEAD` 的图形化版本)

---

#### 4. 比较最新储藏与当前分支
*决定是否要应用（`pop` 或 `apply`）储藏*
```sh
gcmp stash HEAD
```
**它回答了**: "我之前储藏的工作和我当前分支的最新状态有什么不同？应用它是否会产生冲突？"

---

#### 5. 比较两个分支
*Code Review 或合并前的必备操作*
```sh
gcmp main feature-branch
```
**它回答了**: "`feature-branch` 分支相比 `main` 分支，都做了哪些修改？"

---

#### 6. 比较当前状态与某个旧版本
```sh
gcmp work v1.2.0
```
**它回答了**: "我当前的工作相比 `v1.2.0` 版本有哪些变化？"

---

#### 7. 比较两个历史提交
*用于调试代码回归（regression）问题*
```sh
gcmp abc123d def456a
```

---

#### 8. 在指定仓库中进行比较
```sh
gcmp ~/projects/my-app main develop
```

## 🔧 自定义 (Customization)

### 更换比较工具

默认情况下，`gcmp` 使用 `bcompare`。你可以通过编辑脚本顶部的 `DIFF_TOOL` 变量来更换为你喜欢的工具。

```bash
# gcmp.sh

# --- 全局配置 ---
# 你可以在这里修改为你喜欢的比较工具命令
DIFF_TOOL="bcompare" 
```

一些流行的替代品：
*   **Visual Studio Code**: `DIFF_TOOL="code --diff"`
*   **KDiff3**: `DIFF_TOOL="kdiff3"`
*   **Meld**: `DIFF_TOOL="meld"`
*   **P4Merge**: `DIFF_TOOL="p4merge"`

请确保你选择的工具命令在你的系统 `PATH` 中，并且支持接收两个目录路径作为参数进行比较。

## ⚙️ 工作原理 (How It Works)

1.  **解析参数**: 脚本首先解析命令行参数，确定仓库路径、`ref1` 和 `ref2`。
2.  **创建临时环境**: 它会创建一个唯一的临时目录（例如 `/tmp/git-compare-XXXXXX`），并设置一个 `trap` 命令，确保在脚本退出时（无论成功、失败或被中断）这个目录都会被自动删除。
3.  **内容导出**:
    *   对于 `work`，它使用 `git ls-files` 列出所有跟踪的文件，并用 `rsync` 高效地将它们复制到临时子目录中。这能确保只比较 Git 管理的文件。
    *   对于 `stage`，它使用 `git checkout-index`，这是将暂存区内容导出到目录最直接、最准确的方式。
    *   对于所有其他 Git 引用（commit, branch, tag, stash），它使用 `git archive` 将该引用的完整文件树导出到临时子目录中。
4.  **启动比较工具**: 将两个填充好内容的临时子目录路径传递给你配置的 `DIFF_TOOL`。脚本会使用 `-wait` 或类似参数（如果工具支持）等待比较工具关闭。
5.  **自动清理**: 当你关闭比较工具后，脚本继续执行，最终 `trap` 被触发，所有临时文件和目录被彻底删除。

## ⚠️ 依赖与要求 (Dependencies & Requirements)

*   `bash` (v4.0+ 推荐)
*   `git`
*   一个图形化的目录比较工具（例如 [Beyond Compare](https://www.scootersoftware.com/), [VS Code](https://code.visualstudio.com/), [Meld](http://meldmerge.org/),等），并且其可执行文件在你的系统 `PATH` 中。

## 许可 (License)

本项目采用 [MIT License](LICENSE.txt)。