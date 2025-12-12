# Git 常见问题 Git Common Pitfalls

记录使用Git时容易遇到的问题和解决方案。

## 1. 撤销操作 Undoing Changes

### 撤销未暂存的修改
```bash
# 撤销单个文件的修改
git checkout -- <file>
# 或使用新命令
git restore <file>

# 撤销所有未暂存的修改
git checkout -- .
# 或
git restore .
```

### 撤销已暂存的修改
```bash
# 取消暂存单个文件（保留修改）
git reset HEAD <file>
# 或使用新命令
git restore --staged <file>

# 取消暂存所有文件
git reset HEAD
# 或
git restore --staged .
```

### 修改最后一次提交
```bash
# 修改提交信息
git commit --amend -m "新的提交信息"

# 添加遗漏的文件到最后一次提交
git add forgotten_file
git commit --amend --no-edit
```

⚠️ **注意**: `--amend` 会改变提交历史，不要修改已推送的提交！

## 2. 分支管理 Branch Management

### 删除分支的陷阱
```bash
# 删除本地分支（必须先切换到其他分支）
git branch -d branch_name

# 强制删除未合并的分支
git branch -D branch_name

# 删除远程分支
git push origin --delete branch_name
# 或
git push origin :branch_name
```

### 恢复已删除的分支
```bash
# 查找被删除分支的最后一次提交
git reflog

# 恢复分支
git checkout -b recovered_branch <commit-hash>
```

## 3. 合并冲突 Merge Conflicts

### 常见场景
```bash
# 合并时出现冲突
git merge feature_branch
# Auto-merging file.txt
# CONFLICT (content): Merge conflict in file.txt
# Automatic merge failed; fix conflicts and then commit the result.
```

### 解决步骤
```bash
# 1. 查看冲突文件
git status

# 2. 编辑冲突文件，解决冲突标记
# <<<<<<< HEAD
# 当前分支的内容
# =======
# 合并分支的内容
# >>>>>>> feature_branch

# 3. 标记为已解决
git add <resolved-file>

# 4. 完成合并
git commit -m "解决合并冲突"

# 如果想放弃合并
git merge --abort
```

### 使用合并工具
```bash
# 配置合并工具（例如VSCode）
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# 使用合并工具解决冲突
git mergetool
```

## 4. rebase陷阱 Rebase Pitfalls

### 基本使用
```bash
# 将当前分支变基到main
git checkout feature_branch
git rebase main
```

### 常见问题
```bash
# 问题：rebase过程中遇到冲突
# 解决冲突后继续
git add <resolved-file>
git rebase --continue

# 跳过当前提交
git rebase --skip

# 放弃rebase
git rebase --abort
```

### ⚠️ **黄金法则**
**永远不要rebase已经推送到公共仓库的提交！**

```bash
# 错误示例：rebase已推送的分支
git checkout main
git pull
git rebase origin/feature  # 危险！如果其他人也在使用这个分支

# 正确做法：使用merge
git merge origin/feature
```

## 5. .gitignore 不生效 .gitignore Not Working

### 问题
```bash
# 已经被追踪的文件，添加到.gitignore后仍然被追踪
```

### 解决方案
```bash
# 从Git中移除文件但保留在工作目录
git rm --cached <file>
# 或移除整个目录
git rm -r --cached <directory>

# 然后提交
git add .gitignore
git commit -m "更新gitignore"
```

### 常用.gitignore模板
```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
*.egg-info/

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# 项目特定
logs/
*.log
.env
```

## 6. 远程仓库问题 Remote Repository Issues

### 推送被拒绝
```bash
# 错误信息：
# ! [rejected]        main -> main (fetch first)
# error: failed to push some refs to 'remote'

# 原因：远程分支有新提交

# 解决方案1：先拉取再推送
git pull origin main
git push origin main

# 解决方案2：rebase后推送（保持线性历史）
git pull --rebase origin main
git push origin main

# 解决方案3：强制推送（危险！）
git push -f origin main  # 慎用！
```

### 更改远程仓库URL
```bash
# 查看远程仓库
git remote -v

# 修改远程仓库URL
git remote set-url origin <new-url>

# 添加新的远程仓库
git remote add upstream <url>
```

## 7. 大文件问题 Large Files Issues

### 问题
```bash
# 错误：文件太大无法推送
# remote: error: File large_file.zip is 123.45 MB; this exceeds GitHub's file size limit of 100.00 MB
```

### 解决方案

#### 方案1：从历史中移除大文件
```bash
# 使用git filter-branch（老方法）
git filter-branch --tree-filter 'rm -f large_file.zip' HEAD

# 使用BFG Repo-Cleaner（推荐）
# 1. 下载BFG: https://rtyley.github.io/bfg-repo-cleaner/
# 2. 运行清理
java -jar bfg.jar --delete-files large_file.zip
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

#### 方案2：使用Git LFS
```bash
# 安装Git LFS
git lfs install

# 追踪大文件
git lfs track "*.zip"
git lfs track "*.mp4"

# 添加.gitattributes
git add .gitattributes

# 正常提交
git add large_file.zip
git commit -m "添加大文件"
git push
```

## 8. 提交历史问题 Commit History Issues

### 合并多个提交（Squash）
```bash
# 交互式rebase最近3个提交
git rebase -i HEAD~3

# 在编辑器中，将除第一个外的pick改为squash或s
# pick abc1234 第一个提交
# squash def5678 第二个提交
# squash ghi9012 第三个提交
```

### 拆分一个提交
```bash
# 交互式rebase
git rebase -i HEAD~3

# 将要拆分的提交标记为edit
# 当rebase停在该提交时
git reset HEAD^
git add file1
git commit -m "第一部分"
git add file2
git commit -m "第二部分"
git rebase --continue
```

## 9. 子模块问题 Submodule Issues

### 克隆含子模块的仓库
```bash
# 方法1：克隆时初始化子模块
git clone --recursive <repo-url>

# 方法2：克隆后初始化子模块
git clone <repo-url>
git submodule init
git submodule update
# 或简写
git submodule update --init --recursive
```

### 更新子模块
```bash
# 更新所有子模块到远程最新版本
git submodule update --remote

# 更新特定子模块
git submodule update --remote <submodule-path>
```

## 10. 分离HEAD状态 Detached HEAD State

### 问题
```bash
# checkout到特定提交后
git checkout abc1234
# You are in 'detached HEAD' state...
```

### 解决方案
```bash
# 方案1：创建新分支保存当前工作
git checkout -b new_branch

# 方案2：回到某个分支
git checkout main

# 方案3：如果在分离状态做了提交，想保存
git branch temp_branch
git checkout main
git merge temp_branch
```

## 11. 敏感信息泄露 Sensitive Information Leak

### 问题
不小心提交了包含密码、API密钥等敏感信息的文件

### 解决方案
```bash
# ⚠️ 重要：如果已经推送，立即更改密码/密钥！

# 从历史中移除敏感文件
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/sensitive_file" \
  --prune-empty --tag-name-filter cat -- --all

# 强制推送（清除远程历史）
git push origin --force --all
git push origin --force --tags

# 更好的方法：使用BFG
java -jar bfg.jar --delete-files sensitive_file
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push origin --force --all
```

## 最佳实践 Best Practices

1. **频繁提交，完善的提交信息**
```bash
# 好的提交信息
git commit -m "feat: 添加用户认证功能

- 实现JWT token生成
- 添加登录API端点
- 更新用户模型"

# 避免
git commit -m "更新"
```

2. **使用分支进行功能开发**
```bash
# 从main创建功能分支
git checkout -b feature/user-auth main

# 完成后合并
git checkout main
git merge feature/user-auth
git branch -d feature/user-auth
```

3. **定期同步远程仓库**
```bash
# 每天开始工作前
git pull --rebase origin main
```

4. **在推送前检查更改**
```bash
git status
git diff
git log --oneline -5
```

5. **使用.gitconfig设置别名**
```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

## 紧急情况恢复 Emergency Recovery

### 恢复丢失的提交
```bash
# 查看所有操作历史
git reflog

# 恢复到特定状态
git reset --hard <commit-hash>
# 或创建新分支
git checkout -b recovery <commit-hash>
```

### 恢复删除的分支
```bash
# 在reflog中找到分支的最后一次提交
git reflog

# 重新创建分支
git checkout -b recovered_branch <commit-hash>
```

## 标签 Tags
`git` `version-control` `troubleshooting` `常见问题` `版本控制`
