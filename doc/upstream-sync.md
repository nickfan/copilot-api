# Upstream 同步操作指南

本文档记录了从上游仓库（upstream）获取更新并同步到本仓库（origin）的完整流程。本指南适用于 fork 项目需要定期同步上游更新的场景。

## 背景

- **origin**: 本 fork 仓库（nickfan/copilot-api）
- **upstream**: 源仓库（caozhiyuan/copilot-api）
- **主要分支**: `all`（上游主分支）、`myall`（本仓库自定义分支）

## 快速参考命令

```bash
# 步骤1: 更新 all 分支
git checkout all
git stash                     # 暂存本地修改
git fetch upstream           # 拉取上游更新
git merge upstream/all       # 合并上游 all 分支
git push origin all          # 推送到 origin
git stash pop                # 恢复本地修改

# 步骤2: 更新 myall 分支
git checkout myall
ghit stash                   # 暂存本地修改
git fetch origin all         # 拉取已同步的 origin/all
git rebase origin/all        # 变基到最新版本
git push origin myall --force-with-lease  # 强制推送
git stash pop                # 恢复本地修改
```

## 详细操作流程

### 第一部分：同步 all 分支（origin/all ← upstream/all）

#### 1.1 检查当前状态和 remote 配置

```bash
# 查看 remote 配置
git remote -v

# 查看分支列表（确认 all 分支存在）
git branch -a | grep all

# 检查工作区状态
git status
```

**预期输出**：
- upstream: git@github.com:caozhiyuan/copilot-api.git
- origin: git@github.com:nickfan/copilot-api.git
- 当前在 all 分支上有未提交修改

#### 1.2 暂存本地修改

```bash
# 保存工作区修改
git stash
```

**说明**：确保工作区干净，避免合并冲突时丢失本地修改。

#### 1.3 拉取并合并上游更新

```bash
# 拉取 upstream 所有更新
git fetch upstream

# 合并 upstream/all 到本地 all
git merge upstream/all
```

**说明**：使用 `merge` 而非 `rebase` 保持提交历史的线性，因为 all 分支是镜像分支。

**预期输出**：
```
Updating 7c73765..c4a1646
Fast-forward
[文件变更列表]
```

#### 1.4 推送到 origin

```bash
# 推送到 origin/all
git push origin all
```

**预期输出**：
```
To github.com:nickfan/copilot-api.git
   7c73765..c4a1646  all -> all
```

#### 1.5 恢复本地修改

```bash
# 恢复之前暂存的修改
git stash pop
```

**说明**：如果合并过程无冲突，stash 会自动清除。

---

### 第二部分：同步 myall 分支（origin/myall ← origin/all）

#### 2.1 切换到 myall 分支

```bash
# 切换到 myall
git checkout myall

# 查看本地和远程状态
git log --oneline -3
git log --oneline origin/myall -3
```

**说明**：对比本地 myall 和 origin/myall 的差异。

#### 2.2 暂存本地修改

```bash
# 保存工作区修改
git stash
```

**说明**：myall 分支也可能有未完成的工作，需要先暂存。

#### 2.3 拉取最新的 origin/all

```bash
# 拉取已同步的 origin/all
git fetch origin all

# 查看 origin/all 的最新提交
git log --oneline origin/all -3
```

**说明**：此时 origin/all 已与 upstream/all 同步。

#### 2.4 变基到最新版本

```bash
# 将 myall 变基到 origin/all
git rebase origin/all
```

**说明**：使用 `rebase` 而非 `merge` 保持 myall 分支的提交在顶部，历史更清晰。

**预期输出**：
```
Rebasing (1/2)
Rebasing (2/2)
Successfully rebased and updated refs/heads/myall.
```

#### 2.5 强制推送到 origin

```bash
# 强制推送变基后的 myall
git push origin myall --force-with-lease
```

**说明**：
- `--force-with-lease` 比 `--force` 更安全，防止覆盖他人推送的提交
- 由于 rebase 改变了提交历史，必须使用 force push

**预期输出**：
```
To github.com:nickfan/copilot-api.git
 + 21c444b...5c97323 myall -> myall (forced update)
```

#### 2.6 恢复暂存的修改

```bash
# 恢复之前暂存的修改
git stash pop
```

**预期输出**：
```
On branch myall
Your branch is up to date with 'origin/myall'.

Changes not staged for commit:
  modified:   bun.lock
```

---

## 验证操作

### 检查所有分支状态

```bash
# 查看所有分支的同步状态
git branch -vv

# 查看各分支的最新提交
echo "=== upstream/all ===" && git log --oneline upstream/all -3
echo "=== origin/all ===" && git log --oneline origin/all -3
echo "=== local all ===" && git log --oneline all -3
echo "=== origin/myall ===" && git log --oneline origin/myall -3
echo "=== local myall ===" && git log --oneline myall -3

# 查看工作区状态
git status
```

**成功标志**：
1. upstream/all、origin/all、local all 的提交一致
2. origin/myall、local myall 的提交一致
3. myall 分支的提交以 origin/all 的最新提交为基础
4. 工作区保留了之前的未提交修改

---

## 特殊情况处理

### 情况1：合并冲突

如果在 `git merge upstream/all` 或 `git rebase origin/all` 时出现冲突：

```bash
# 查看冲突文件
git status

# 手动解决冲突（编辑冲突文件）
# 标记已解决
git add <conflicted-file>

# 继续合并或变基
git merge --continue    # 或 git rebase --continue

# 如果无法解决，中止操作
git merge --abort       # 或 git rebase --abort
```

### 情况2：远程分支已被他人更新

如果 push 失败：

```bash
# 先拉取最新变更（对于非 rebase 分支）
git pull --rebase origin all

# 然后再次 push
git push origin all

# 如果 myall push 失败（因为其他人也 force push）
# 需要重新评估，可能需要重新变基
git fetch origin
git rebase origin/myall
git push origin myall --force-with-lease
```

### 情况3：Stash 冲突

如果 `stash pop` 时出现冲突：

```bash
# 查看冲突
git status

# 解决冲突后
git add .
git stash drop  # 手动删除 stash（如果 git stash pop 没有自动删除）
```

---

## 最佳实践

1. **定期同步**：建议每周至少同步一次上游更新，避免差异过大
   
2. **提交前同步**：在提交 PR 或开始新功能前，先同步上游更新

3. **使用 stash**：始终在操作前暂存工作区修改，防止意外丢失

4. **验证同步**：每次同步后验证分支状态，确保成功

5. **选择合适策略**：
   - 镜像分支（all）用 merge，保持与原仓库一致
   - 自定义分支（myall）用 rebase，保持历史清晰

6. **安全强制推送**：始终使用 `--force-with-lease` 而非 `--force`

7. **备份重要分支**：在重要分支做破坏性操作前，先创建备份分支
   ```bash
   git checkout myall
   git branch myall-backup-$(date +%Y%m%d)  # 创建备份
   ```

---

## 常见问题 FAQ

**Q: 为什么要两次 stash？**
A: 每个分支的变更需要独立处理。all 和 myall 都可能有未提交修改。

**Q: 为什么 all 用 merge，myall 用 rebase？**
A: all 是镜像分支，应与上游保持一致；myall 是自定义分支，rebase 保持提交历史线性。

**Q: 可以简化为一步吗？**
A: 不建议。每次处理后验证状态，更容易发现和解决问题。

**Q: 如果只有一个分支需要更新怎么办？**
A: 按需执行对应步骤，跳过不需要的分支。

---

## 总结

本流程确保：

1. ✅ **origin/all** 与 **upstream/all** 保持同步
2. ✅ **myall** 分支基于最新的上游更新
3. ✅ **origin/myall** 与本地 myall 保持同步
4. ✅ **工作区修改** 始终得到保存和恢复
5. ✅ **提交历史** 清晰且可追溯

---

## 最后更新

- **创建日期**: 2025-12-11
- **创建者**: AI Assistant
- **相关项目**: copilot-api
- **Git 配置**: 
  - upstream: caozhiyuan/copilot-api
  - origin: nickfan/copilot-api

---

## 附录：PowerShell 注意事项

如果使用 PowerShell，命令中的 `&&` 不能直接使用，需要改用分号 `;` 或分行执行。

```powershell
# ❌ 错误
command1 && command2 && command3

# ✅ 正确（使用 Out-Host）
command1 | Out-Host; command2 | Out-Host; command3

# ✅ 正确（分行）
command1
command2
command3
```
