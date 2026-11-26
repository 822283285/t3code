# 本 Fork 的开发流程

适用于 [822283285/t3code](https://github.com/822283285/t3code)，本地目录为 `/Users/a0000/workspace/mine-project/t3code`。本文同时约束人工操作与 AI，用户在当前任务中明确指定的范围优先。

## 远程仓库与分支

Fork 是 GitHub 上属于自己的仓库；`origin`、`upstream` 是远程仓库别名，不是分支。

| 远程       | 地址                                      | 用途                         |
| ---------- | ----------------------------------------- | ---------------------------- |
| `upstream` | `https://github.com/pingdotgg/t3code.git` | 获取官方更新，不向它推送     |
| `origin`   | `https://github.com/822283285/t3code.git` | 保存自己的官方镜像和定制代码 |

| 分支                        | 职责                         | 代码来源与去向                                            |
| --------------------------- | ---------------------------- | --------------------------------------------------------- |
| `main`                      | 官方代码镜像，不添加定制提交 | 从 `upstream/main` 快进更新，推送到 `origin/main`         |
| `custom`                    | 长期定制开发与集成           | 拉取 `origin/custom`，合入 `main`，推送到 `origin/custom` |
| `feat/<主题>`、`fix/<主题>` | 可选的独立任务分支           | 从 `custom` 创建，验证后合回 `custom`                     |

简单修改直接在 `custom` 开发，不强制创建任务分支。不额外维护长期发布分支；从 `custom` 选择通过验收的提交打包，用 `custom-YYYYMMDD-N` 标签记录成功安装的版本。

## 首次配置与推送 custom

首次使用一份检出目录时，先确认 `origin` 能抓取定制分支。本地原先仅抓取 `main` 时，恢复为抓取 Fork 的全部分支：

```sh
git remote set-branches origin '*'
```

若 Fork 尚无 `custom`，首次获授权推送定制代码时执行：

```sh
git push -u origin custom
```

推送会创建远程 `custom` 并设置本地跟踪关系。写入本规范不代表已经发布定制分支。

## 日常开发与拉取 Fork

1. 检查 `git status --short --branch`，确认现有改动及其归属，保护尚未提交的工作。
2. 在 `custom` 接收自己在其他机器或会话中推送的定制代码：

   ```sh
   git switch custom
   git pull --ff-only origin custom
   ```

   远程 `custom` 尚未创建时跳过拉取。若双方都有新提交导致无法快进，检查差异后正常合并，不丢弃任一侧工作。

3. 在 `custom` 修改代码；独立任务需要隔离时从它创建 `feat/<主题>` 或 `fix/<主题>`，完成后合回 `custom`。不要从官方 `main` 开始一个依赖现有定制的功能。
4. 用项目的“开发 Web”或“开发 Desktop”脚本验证，按改动范围运行必要检查。推送前执行“检查改动”，并核对 `git diff --check` 和工作区状态。
5. 按业务行为或职责提交，使用 `feat` / `fix` / `chore` / `refactor` / `docs` 类型和中文概要，必要的正文使用 `[新增/修复/优化/重构/文档]` 中对应的单一标签说明具体变化。获授权推送时推送到 `origin/custom`；功能分支的推送和 PR 依任务授权处理。

“从 Fork 拉代码”是在 `custom` 拉 `origin/custom`；“获取官方更新”走下一节。把定制合回官方镜像 `main` 会破坏这两个职责的分离。

## 同步官方代码

用户要求“同步一次代码”时，默认完成本节的本地和 Fork 同步，不安装或替换正式客户端。明确要求只在本地操作时不推送。

1. 检查工作区、远程地址和分支历史，保护已有工作。确认 `main` 与 `origin/main` 没有夹带定制提交；若有，先查明并保留，不能强制覆盖。
2. 获取官方和 Fork 的更新，更新官方镜像，再合入定制分支：

   ```sh
   git fetch upstream main
   git fetch origin
   git switch main
   git merge --ff-only upstream/main
   git switch custom
   # 远程 custom 已存在时，先接收它的更新。
   git merge --ff-only origin/custom
   git merge main -m "chore: 同步官方代码"
   ```

   首次没有 `origin/custom` 时跳过对应合并。任何快进失败或合并冲突都要先检查差异；定制分支使用 merge 保留历史，不 rebase 已发布的 `custom`，不以 reset 或强制推送替代冲突处理。业务含义无法确认时说明具体冲突并等待答案，不猜测取舍。

3. 锁文件或依赖配置有变化时，使用项目“安装依赖”脚本按锁文件更新依赖。完成受影响检查、项目“检查改动”和 `git diff --check`，修复本次合并引入的问题，不擅自扩大到全仓库检查。
4. 检查通过后推送 `main` 和 `custom`。首次推送 `custom` 时按前文设置抓取范围和跟踪关系：

   ```sh
   git push origin main
   git push -u origin custom
   ```

5. 保持当前分支为 `custom`，核对远程提交。交付说明官方基线、定制分支提交、冲突处理及验证结果。`custom` 存在定制提交时，它本就可以领先于 `main`，不能为了让两者 SHA 一致而抹掉定制。

## 验证与发布自用版本

同步源码不会改变正在使用的已安装客户端。发布从 `custom` 的一个已提交、已验收版本开始：

1. 记录目标提交 SHA，确认需要发布的定制均已包含，并完成受影响功能的验收。
2. 使用项目开发脚本验证。当前脚本固定使用 Node `26.13.1`、pnpm `12.14.0` 和隔离数据目录 `/Users/a0000/.t3/dev-t3code`；不得把开发服务指向正式目录 `~/.t3/userdata`。
3. 用户要求构建 macOS 安装包时，在目标提交构建：

   ```sh
   /Users/a0000/.vite-plus/bin/vp env exec --node 26.13.1 --package-manager pnpm@12.14.0 vp run dist:desktop:dmg
   ```

   产物位于 `release/`，构建成功只表示得到安装包。构建前核对工具版本是否仍与 `package.json` 一致；其他平台及构建前置条件见 [开发环境](00_development.md#desktop-artifacts)。

4. 仅在安装或发布已经获授权时替换正式客户端。保留旧安装包及正式状态备份，由应用执行正常的数据迁移，不直接改写正式数据库。若替换会影响当前承载会话的 T3 Code，先交付可核对的安装包及切换步骤，再安排切换。
5. 实际安装并验收成功后，在实际构建的提交上创建唯一标签，例如 `custom-20261126-1`；按发布授权推送这一个标签。标签不可覆盖或移动，不把安装前的候选版本标为已上线。

后续自用版本沿用“同步官方 → 合入 custom → 验证 → 构建 → 安装验收 → 标记版本”。需要回退时指定既有标签及安装包，先核对数据兼容性；不要为了回退客户端重置 `main`、`custom` 或移动旧标签。

AI 的交付说明必须区分代码同步、依赖更新、构建、安装和实际功能验收，列出未完成项，不把任何前一步当作后一步完成。
