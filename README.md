
English | [简体中文](README_CN.md)
# gcmp - Git Compare on Steroids

`gcmp` (Git Compare) is a powerful command-line script that leverages Beyond Compare (or other graphical diff tools) to visually compare any two "areas" within a Git repository. It seamlessly combines the flexibility of `git diff` with the user-friendliness of a GUI tool, dramatically improving the efficiency of code reviews and change analysis.

<!-- A short GIF demonstrating the script in action would be a great addition here! -->

---

## ✨ Core Features

*   **Intuitive Keywords**: Use simple keywords like `work`, `stage`, and `stash` to represent the working directory, staging area, and the latest stash.
*   **Full-Spectrum Comparison**: Supports comparison between the **Working Directory**, **Staging Area**, **Stash**, and any **commit, branch, or tag**.
*   **Live Editing & Performance**: When comparing the `work` directory, `gcmp` uses your actual project folder. This allows you to **edit files directly in your diff tool** and enjoy instant comparisons without any copying overhead.
*   **Smart Auto-Upgrading**: The script is smart! If you provide a reference like `HEAD` or a branch name, and it matches your clean working directory, `gcmp` automatically uses the live directory for comparison. You get the performance and editing benefits without explicitly typing `work`.
*   **Highly Customizable**: Easily switch the default `bcompare` to your favorite diff tool (like `kdiff3`, `meld`, `vscode`, etc.).
*   **Safe and Reliable**: References that don't qualify for live comparison are safely exported to temporary directories that are automatically cleaned up on exit, protecting your repository's state.

## ❓ Why use `gcmp`?

Standard `git diff` is powerful, but its terminal-based, file-by-file output can make it difficult to grasp the big picture for large or scattered changes.

`gcmp` solves the following pain points:

1.  **The Big Picture**: Want to know exactly which files were changed, added, or deleted in `feature-A` compared to `main`? `gcmp` presents this clearly as a directory tree.
2.  **Simplified Complex Comparisons**: An operation like `git diff work stage` doesn't exist natively. But `gcmp work stage` makes it trivial, clearly showing which changes in your working directory have not yet been staged.
3.  **GUI Advantage**: For complex changes within a single file, a GUI tool's side-by-side scrolling, syntax highlighting, and inline diffs are far more intuitive than the `+` and `-` lines in a terminal.

## 🚀 Installation

We recommend placing the `gcmp` script in your system's `PATH` for easy access from any repository.

### Manual Install

1.  Download `gcmp`.
2.  Give it execute permissions in your terminal: `chmod +x gcmp`.
3.  Move this file to a directory included in your `PATH` environment variable, such as `~/bin` or `/usr/local/bin`.

## 💡 Usage

### Basic Syntax

```
gcmp [repo_path] <ref1> <ref2>
```

*   `repo_path` (Optional): The path to the Git repository. If omitted, the current directory is used.
*   `<ref1>` & `<ref2>`: The two Git references or keywords to compare.

### Special Keywords

This is the core magic of `gcmp`, making complex comparisons simple.

| Keyword | Represents Git Area | Description                                                  |
|:--------|:--------------------|:-------------------------------------------------------------|
| `work`  | Working Directory   | All tracked files as they currently exist. This is a **live view**¡ªchanges you make in the diff tool are saved directly to your files! |
| `stage` | Staging Area (Index)| Content that has been added with `git add` for the next commit. |
| `stash` | The Latest Stash    | Represents `stash@{0}`, your most recent stash.              |

In addition to these keywords, `<ref1>` and `<ref2>` can be any reference Git understands (a "commit-ish"), such as:
*   **Commit HASH**: `abc123d`, `7a4e3f81`
*   **Branch Name**: `main`, `my-feature`
*   **Tag Name**: `v1.0.2`
*   **Relative Ref**: `HEAD`, `HEAD~2`, `main~1`
*   **Other Stashes**: `'stash@{1}'` (note the quotes)

### Common Examples

---

#### 1. Compare Working Directory vs. Staging Area
*See what changes haven't been staged yet*
```sh
gcmp work stage
```
**It answers:** "What changes will be added to the index if I run `git add .` right now?"

**Pro-Tip**: You can edit your files on the `work` side of the comparison, save them, and they are immediately updated in your actual working directory, ready for you to `git add`.

---

#### 2. Compare Staging Area vs. Last Commit
*Preview what you are about to commit*
```sh
gcmp stage HEAD
```
**It answers:** "What will the snapshot look like if I run `git commit` right now?" (This is the graphical equivalent of `git diff --staged`)

**Pro-Tip**: On a clean repository, you can run `gcmp HEAD HEAD~1`. `gcmp` will automatically use your live working directory for `HEAD`, giving you the fastest possible comparison. <!-- ADDED -->

---

#### 3. Compare Working Directory vs. Last Commit
*See all local changes since the last commit (both staged and unstaged)*
```sh
gcmp work HEAD
```
**It answers:** "What have I changed since my last commit?" (This is the graphical equivalent of `git diff HEAD`)

---

#### 4. Compare Latest Stash vs. Current Branch
*Decide whether to `pop` or `apply` a stash*
```sh
gcmp stash HEAD
```
**It answers:** "How does my stashed work differ from the current state of my branch? Will applying it cause major conflicts?"

---

#### 5. Compare Two Branches
*Essential for code reviews or pre-merge checks*
```sh
gcmp main feature-branch
```
**It answers:** "What changes has `feature-branch` introduced compared to `main`?"

---

#### 6. Compare Current State vs. an Old Version
```sh
gcmp work v1.2.0
```
**It answers:** "How does my current work differ from the `v1.2.0` release?"

---

#### 7. Compare Two Historical Commits
*Useful for debugging a code regression*
```sh
gcmp abc123d def456a
```

---

#### 8. Compare Within a Specific Repository
```sh
gcmp ~/projects/my-app main develop
```

## 🔧 Customization

### Changing the Diff Tool

By default, `gcmp` uses `bcompare`. You can change this to your preferred tool by editing the `DIFF_TOOL` variable at the top of the script.

```bash
# gcmp.sh

# --- Global Configuration ---
# You can change this to your favorite diff tool command
DIFF_TOOL="bcompare" 
```

Some popular alternatives:
*   **Visual Studio Code**: `DIFF_TOOL="code --diff"`
*   **KDiff3**: `DIFF_TOOL="kdiff3"`
*   **Meld**: `DIFF_TOOL="meld"`
*   **P4Merge**: `DIFF_TOOL="p4merge"`

Ensure your chosen tool's command is in your system `PATH` and that it supports receiving two directory paths as arguments for comparison.

## ⚙️ How It Works

1.  **Parse Arguments**: The script first determines the repository path and the two references (`ref1` and `ref2`) from the command-line arguments.
2.  **Prepare Comparison Environment**: It creates a unique temporary parent directory and sets a `trap` command to ensure this directory is automatically deleted when the script exits.
3.  **Intelligently Resolve Sources**: For each of the two references, the script decides the best way to present it:
    *   **Live Directory**: If the reference is the `work` keyword, OR if it's a reference (like `HEAD`) that matches a clean working directory, the script uses the **direct path to your repository**. This is the fastest method and enables live editing.
    *   **Temporary Snapshot**: If a reference doesn't qualify for live comparison (e.g., it's an old commit, `stage`, or `stash`), it is exported to a temporary subdirectory using the most appropriate Git command (`git checkout-index` for `stage`, `git archive` for others).
4.  **Launch Diff Tool**: Before launching, it performs a final check to ensure the two resolved paths are not identical (preventing self-comparison). It then passes the paths for each side to your configured `DIFF_TOOL`, using a `-wait` flag to pause until you close the tool.
5.  **Automatic Cleanup**: Once you close the diff tool, the script resumes, and the `trap` command is triggered on exit, completely removing any temporary directories created during the process.

## ⚠️ Dependencies & Requirements

*   `bash` (v4.0+ recommended)
*   `git`
*   A graphical directory comparison tool (e.g., [Beyond Compare](https://www.scootersoftware.com/), [VS Code](https://code.visualstudio.com/), [Meld](http://meldmerge.org/), etc.) with its executable in your system `PATH`.

## 📜 License

This project is licensed under the [MIT License](LICENSE.txt).