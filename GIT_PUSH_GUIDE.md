# Git 推送操作指南

## 快速推送流程

### 方法 1: 标准推送流程（推荐）

```bash
# 1. 查看当前状态
git status

# 2. 添加新文件或修改的文件
git add <文件名>           # 添加单个文件
git add .                  # 添加所有修改的文件

# 3. 创建提交
git commit -m "描述你的更改"

# 4. 推送到 GitHub
git push origin main
```

### 方法 2: 一步推送（适合小改动）

```bash
# 添加、提交、推送一步完成
git add . && git commit -m "更新内容" && git push origin main
```

---

## 详细操作步骤

### 第一步：查看当前状态

```bash
# 查看哪些文件被修改了
git status

# 查看具体修改内容
git diff
```

### 第二步：添加文件到暂存区

```bash
# 添加单个文件
git add README.md

# 添加多个文件
git add file1.md file2.md

# 添加所有修改的文件
git add .

# 添加所有文件（包括删除的文件）
git add -A

# 交互式添加（可以选择部分修改）
git add -p
```

### 第三步：创建提交

```bash
# 基本提交
git commit -m "添加新功能"

# 详细提交信息
git commit -m "标题" -m "详细描述"

# 提交并显示差异
git commit -v

# 修改上一次提交（未推送时）
git commit --amend -m "新的提交信息"
```

**提交信息最佳实践**:
```
好的提交信息格式:
- 添加用户登录功能
- 修复登录页面的样式问题
- 更新 README 文档
- 重构数据库连接代码

不好的提交信息:
- update
- fix
- changes
- ...
```

### 第四步：推送到 GitHub

```bash
# 推送到 main 分支
git push origin main

# 首次推送并设置上游分支
git push -u origin main

# 推送所有分支
git push --all origin

# 强制推送（谨慎使用）
git push -f origin main
```

---

## 常见场景操作

### 场景 1: 添加新文件

```bash
# 创建新文件
echo "# 新文档" > new-doc.md

# 添加到 Git
git add new-doc.md

# 提交
git commit -m "添加新文档"

# 推送
git push origin main
```

### 场景 2: 修改现有文件

```bash
# 修改文件
vim README.md

# 查看修改
git diff README.md

# 添加修改
git add README.md

# 提交
git commit -m "更新 README 文档"

# 推送
git push origin main
```

### 场景 3: 删除文件

```bash
# 删除文件
git rm old-file.md

# 或者先删除文件，再添加到 Git
rm old-file.md
git add old-file.md

# 提交
git commit -m "删除旧文件"

# 推送
git push origin main
```

### 场景 4: 重命名文件

```bash
# 重命名文件
git mv old-name.md new-name.md

# 提交
git commit -m "重命名文件"

# 推送
git push origin main
```

### 场景 5: 撤销未提交的修改

```bash
# 撤销工作区的修改
git restore <文件名>

# 撤销暂存区的修改
git restore --staged <文件名>

# 撤销所有修改
git restore .

# 撤销最近的提交（保留修改）
git reset --soft HEAD~1

# 撤销最近的提交（丢弃修改）
git reset --hard HEAD~1
```

---

## 分支操作

### 创建和切换分支

```bash
# 创建新分支
git branch feature-branch

# 切换到新分支
git checkout feature-branch

# 创建并切换（推荐）
git checkout -b feature-branch

# 推送新分支到 GitHub
git push -u origin feature-branch
```

### 合并分支

```bash
# 切换到主分支
git checkout main

# 合并 feature-branch 到 main
git merge feature-branch

# 推送合并结果
git push origin main

# 删除已合并的分支
git branch -d feature-branch
git push origin --delete feature-branch
```

---

## 同步远程仓库

### 拉取最新更改

```bash
# 拉取并合并
git pull origin main

# 仅拉取不合并
git fetch origin

# 查看远程更新
git log origin/main

# 合并远程更新
git merge origin/main
```

### 解决冲突

```bash
# 拉取时发现冲突
git pull origin main

# 查看冲突文件
git status

# 手动解决冲突（编辑冲突文件）
# 冲突标记:
# <<<<<<< HEAD
# 本地修改
# =======
# 远程修改
# >>>>>>> origin/main

# 解决后添加文件
git add <冲突文件>

# 提交合并
git commit -m "解决合并冲突"

# 推送
git push origin main
```

---

## 查看历史和状态

### 查看提交历史

```bash
# 查看提交历史
git log

# 简洁模式
git log --oneline

# 图形化显示
git log --oneline --graph --all

# 查看最近 5 次提交
git log -5

# 查看某个文件的修改历史
git log --follow <文件名>
```

### 查看差异

```bash
# 查看工作区和暂存区的差异
git diff

# 查看暂存区和最新提交的差异
git diff --staged

# 查看两个提交之间的差异
git diff <commit1> <commit2>

# 查看某个文件的修改
git diff <文件名>
```

### 查看远程仓库信息

```bash
# 查看远程仓库
git remote -v

# 查看远程分支
git branch -r

# 查看所有分支
git branch -a
```

---

## 使用 Personal Access Token 推送

### 配置令牌

```bash
# 方法 1: 在推送时输入令牌
git push origin main
# Username: alanvps2017
# Password: ghp_xxxxxxxxxxxx

# 方法 2: 在 URL 中嵌入令牌（不推荐）
git remote set-url origin https://alanvps2017:ghp_xxx@github.com/alanvps2017/github-dcn.git

# 方法 3: 使用 Git Credential Helper（推荐）
git config --global credential.helper store
# 下次推送时输入一次令牌，之后会自动保存
```

### macOS 钥匙串存储

```bash
# 配置使用 macOS 钥匙串
git config --global credential.helper osxkeychain

# 推送时输入令牌
git push origin main
# Username: alanvps2017
# Password: ghp_xxxxxxxxxxxx
# 之后会自动保存到钥匙串
```

---

## 完整工作流程示例

### 示例 1: 添加新文档

```bash
# 1. 创建新文档
cat > NEW_GUIDE.md << 'EOF'
# 新指南

这是新指南的内容。
EOF

# 2. 查看状态
git status

# 3. 添加文件
git add NEW_GUIDE.md

# 4. 提交
git commit -m "添加新指南文档"

# 5. 推送
git push origin main
```

### 示例 2: 更新多个文件

```bash
# 1. 修改多个文件
vim README.md
vim DOCKER_GUIDE.md

# 2. 查看修改
git status
git diff

# 3. 添加所有修改
git add .

# 4. 提交
git commit -m "更新 README 和 Docker 指南"

# 5. 推送
git push origin main
```

### 示例 3: 使用分支开发新功能

```bash
# 1. 创建并切换到新分支
git checkout -b feature/new-guide

# 2. 在新分支上工作
vim NEW_FEATURE.md
git add NEW_FEATURE.md
git commit -m "添加新功能文档"

# 3. 推送新分支到 GitHub
git push -u origin feature/new-guide

# 4. 切换回主分支
git checkout main

# 5. 合并新分支
git merge feature/new-guide

# 6. 推送主分支
git push origin main

# 7. 删除本地分支
git branch -d feature/new-guide

# 8. 删除远程分支
git push origin --delete feature/new-guide
```

---

## 常见问题解决

### 问题 1: 推送被拒绝

```bash
# 错误: ! [rejected] main -> main (fetch first)
# 原因: 远程仓库有新的提交

# 解决方法:
git pull origin main
git push origin main

# 或者使用 rebase
git pull --rebase origin main
git push origin main
```

### 问题 2: 忘记添加文件到提交

```bash
# 修改上一次提交（未推送时）
git add <忘记的文件>
git commit --amend --no-edit
git push origin main
```

### 问题 3: 推送了错误的提交

```bash
# 撤销最近的提交（未推送时）
git reset --soft HEAD~1

# 修改后重新提交
git add .
git commit -m "正确的提交信息"
git push origin main
```

### 问题 4: 令牌过期

```bash
# 创建新令牌: https://github.com/settings/tokens/new
# 更新凭据
git config --global credential.helper store
git push origin main
# 输入新令牌
```

### 问题 5: 文件太大无法推送

```bash
# 错误: File is larger than 100MB
# 解决方法:

# 1. 使用 Git LFS
git lfs install
git lfs track "*.pdf"
git add .gitattributes
git add large-file.pdf
git commit -m "添加大文件"
git push origin main

# 2. 或者压缩文件
zip -r large-file.zip large-file.pdf
git add large-file.zip
git commit -m "添加压缩文件"
git push origin main
```

---

## 最佳实践

### 1. 提交频率
- ✅ 小步快跑：频繁提交小的改动
- ❌ 一次性提交大量改动

### 2. 提交信息
- ✅ 清晰描述改动内容
- ❌ 模糊的提交信息

### 3. 推送前检查
```bash
# 推送前检查
git status          # 查看状态
git diff            # 查看修改
git log --oneline -5  # 查看最近提交
```

### 4. 定期同步
```bash
# 开始工作前拉取最新代码
git pull origin main

# 工作完成后推送
git push origin main
```

### 5. 使用 .gitignore
```bash
# 创建 .gitignore 文件
cat > .gitignore << 'EOF'
# 操作系统文件
.DS_Store
Thumbs.db

# 编辑器文件
.vscode/
.idea/
*.swp
*.swo

# 依赖目录
node_modules/
vendor/

# 编译文件
*.o
*.exe
*.out

# 日志文件
*.log

# 环境变量
.env
.env.local
EOF

git add .gitignore
git commit -m "添加 .gitignore 文件"
git push origin main
```

---

## 快速参考命令

```bash
# 基本操作
git status              # 查看状态
git add .               # 添加所有修改
git commit -m "msg"     # 提交
git push origin main    # 推送

# 分支操作
git branch              # 查看分支
git checkout -b new     # 创建并切换分支
git merge branch-name   # 合并分支

# 同步操作
git pull origin main    # 拉取更新
git fetch origin        # 获取远程信息

# 撤销操作
git restore <file>      # 撤销修改
git reset --soft HEAD~1 # 撤销提交

# 查看历史
git log --oneline       # 查看历史
git diff                # 查看差异
```

---

## 相关资源

- [Git 官方文档](https://git-scm.com/doc)
- [GitHub 文档](https://docs.github.com)
- [Pro Git 书籍](https://git-scm.com/book/zh/v2)
- [GitHub Skills](https://skills.github.com/)

---

## 总结

推送新内容到 GitHub 的标准流程:

1. **修改文件** - 创建或编辑文件
2. **查看状态** - `git status`
3. **添加文件** - `git add .`
4. **创建提交** - `git commit -m "描述"`
5. **推送到 GitHub** - `git push origin main`

记住这个流程，你就可以轻松地推送新内容到 GitHub 仓库了！
