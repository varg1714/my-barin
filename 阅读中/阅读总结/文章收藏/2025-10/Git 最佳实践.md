---
source:
  - "[[Git 最佳实践，这样用就对了]]"
create: 2025-10-28
---

## 1. Git 的本质：理解核心概念

* **Git 是什么**：一个**分布式**、基于**目录**的版本控制系统，其工作流是**非线性**的。每个本地克隆都是一个包含完整历史的完整仓库。
* **Git 不是什么**：
    * **GitHub/GitLab**：这些是提供 Git 托管和协作流程的**服务网站**，不是 Git 本身。
    * **Fork**：网站功能，本质是克隆一份服务端仓库并设置好上游，方便发起 Pull Request，并非 Git 的原生命令。
    * **Pull/Merge Request**：网站提供的 Code Review 和合并流程，不是 Git 的一部分。在本地合并自己的分支无需创建 MR。

## 2. 分支策略：清晰管理项目脉络

清晰的分支策略是高效协作的基础。

| 分支类型           | 示例                | 用途和生命周期                                                                                |
| :------------- | :---------------- | :------------------------------------------------------------------------------------- |
| **Main 分支** | (`main` / `master`) | **稳定分支**。只包含经过完整测试、可随时发布的代码。                                                           |
| **Develop 分支** | (`develop`)       | **开发主干**。所有新功能开发的汇集点，是功能分支的父分支。                                                        |
| **Feature 分支** | (`feature/xxx`)   | **功能分支**。生命周期短，专注于单个特性开发。从 `develop` 创建，完成后合并回 `develop`，然后**立即删除**。                   |
| **Release 分支** | (`release/vx.x`)  | **发布分支**。用于准备一个新版本的发布，通常从 `develop` 或 `main` 分叉。Bug 修复在主干完成后，通过 `cherry-pick` 同步到发布分支。 |

## 3. 核心操作对比 (1)：`Merge` vs. `Rebase`

这是 Git 中最关键也最容易混淆的概念，核心区别在于**是否重写历史**。

| 特性       | Merge (合并)                                                           | Rebase (变基/嫁接)                                                            |
| :------- | :------------------------------------------------------------------- | :------------------------------------------------------------------------ |
| **工作方式** | 将两个分支的差异进行三方合并，并**创建一个全新的 "merge commit"** 来记录这次合并。这个 commit 有两个父节点。 | 将当前分支的提交“剪切”下来，然后“嫁接”到目标分支的最新提交之后。它通过**重放（Replay）**来**重写提交历史**。           |
| **历史记录** | **非线性**。保留了分支的完整分叉、合并历史，忠实记录了发生过的一切。                                 | **线性**。创造出一条干净的、单一的提交线，看起来就像所有开发都是顺序进行的。                                  |
| **优点** | 真实、可追溯，保留了分支开发的上下文。                                                  | 历史记录非常干净、清晰，没有多余的合并信息。                                                    |
| **缺点** | 分支多时，历史图会变得非常杂乱，难以阅读。                                                | 重写了历史，不应在已被共享的公共分支上使用。                                                    |
| **适用场景** | **公共分支的合并**。例如将 `develop` 合并到 `main`，需要明确记录这个重要的集成点。                 | **个人开发分支的维护**。例如在 `feature` 分支上同步 `develop` 的最新代码，或在提交 PR 前整理自己的 commits。 |

**关键理解**：当你在 `feature` 分支上执行 `git rebase develop` 时，**只有 `feature` 分支会被改变**，`develop` 分支不受任何影响。这个操作的目的是让你的 `feature` 分支“跟上” `develop` 的最新进度。

![Merge GIF](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvavGakaVxzVI9nT1Yk61A8tAJ9p8rNt8ajWkzEKtAT2icNUT8cVicKPUqUCXLMA6NLHqrAPj1pgnzfRQ/640?wx_fmt=gif&from=appmsg)

![Rebase GIF](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvavGakaVxzVI9nT1Yk61A8tAkwTUQ4Wa7NxIFHnxobD12TBrISDg9LX2VePFKuiaZWkxD5fXsKNrCJQ/640?wx_fmt=gif&from=appmsg)

## 4. 核心操作对比 (2)：`Pull` vs. `Fetch`

`git pull` 是一个自动化命令，常常导致不必要的历史混乱。

| 命令              | 做了什么                      | 结果                                                  | 控制力   |
| :-------------- | :------------------------ | :-------------------------------------------------- | :---- |
| **`git pull`** | `git fetch` + `git merge` | 自动下载并合并。如果本地和远程都有新提交，会**自动创建一个 merge commit**，弄乱历史。 | **低** |
| **`git fetch`** | 只下载远程更新                   | **只更新本地的远程分支记录**（如 `origin/main`），不改变你当前的工作分支，非常安全。 | **高** |

**最佳实践（推荐的同步工作流）：**

1. **`git fetch`**：安全地获取远程所有分支的最新更新。
2. **`git rebase origin/develop`**：在你的 `feature` 分支上，将你的本地提交“嫁接”到最新的远程 `develop` 分支之上。这能保持历史线性，并让你在本地提前解决冲突。

## 5. Commit 与 LFS 最佳实践

* **小而完整的 Commit**：
    * **原子性**：每个 commit 都应是一个独立、完整且可工作的单元。
    * **避免一个 commit 做多件事**：如果一个 commit 包含多个不相关的修改，应将其拆分。
    * **避免多个不完整的 commit**：开发过程中的零碎提交（如 "wip", "fix typo"）在发起 MR 前，应使用**交互式 Rebase (`git rebase -i`)** 将它们 `squash`（压缩）成一个或几个有意义的 commit。
    * **`commit --amend`**：在开发单个功能时，对于微小的修改，可持续使用 `amend` 来更新上一个 commit，保持本地历史的整洁。
* **LFS (Large File Storage) 技巧**：
    * **作用**：用于追踪大的二进制文件，避免 Git 仓库体积无限膨胀。
    * **正确用法**：**在添加大文件之前**，必须先将其路径或通配符添加到 `.gitattributes` 文件中。
    * **常见错误**：
        1. **忘记配置 LFS**：先提交了大文件，再配置 `.gitattributes`。这会导致大文件永久留在 Git 的历史记录中。
        2. **滥用 LFS**：将所有文件都用 LFS 追踪，会使 Git 丧失大部分优势，效率急剧下降。

## 6. 总结：一个高效的 Git 工作流

1. **开始新功能**：
    * `git fetch origin`
    * `git checkout develop`
    * `git rebase origin/develop` (确保本地 develop 是最新的)
    * `git checkout -b feature/my-new-feature`

2. **开发过程中**：
    * 进行修改，然后 `git add .`
    * `git commit -m "feat: implement X feature"`
    * 对于后续的小修改，使用 `git commit --amend` 来更新上一个提交，保持本地历史干净。

3. **与主干保持同步**：
    * 定期执行 `git fetch origin`
    * `git rebase origin/develop` 将你的功能分支嫁接到最新的 `develop` 分支上，并解决可能出现的冲突。

4. **准备提交合并请求 (MR/PR)**：
    * 使用 `git rebase -i origin/develop`，进入交互模式。
    * 将开发过程中的多个零碎 commit 通过 `squash` 或 `fixup` 合并成一个或少数几个逻辑清晰的 commit，并写好 commit message。

5. **提交与清理**：
    * `git push origin feature/my-new-feature --force-with-lease` (因为 rebase 重写了历史，需要强制推送，`--force-with-lease` 是更安全的选择)。
    * 在 GitLab/GitHub 上创建 Merge Request。
    * MR 被合并后，**删除远程和本地的功能分支**。

遵循这套流程，你的 Git 历史将始终保持清晰、线性和专业，极大地提升个人效率和团队协作体验。