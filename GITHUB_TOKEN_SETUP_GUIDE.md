# GitHub Personal Access Token 设置指南

## 什么是 Personal Access Token？

Personal Access Token (PAT) 是 GitHub 提供的一种安全认证方式，用于替代密码进行 Git 操作和 API 访问。GitHub 已于 2021 年 8 月停止支持密码认证，现在必须使用 PAT 或 SSH 密钥。

---

## 详细设置步骤

### 第一步：登录 GitHub

1. 打开浏览器，访问 [GitHub](https://github.com)
2. 使用您的账号登录（用户名：wlxx2017）

### 第二步：进入 Settings 页面

有两种方式进入 Settings：

**方式 1：通过右上角菜单**
1. 点击右上角的头像图标
2. 在下拉菜单中选择 **Settings**（设置）

**方式 2：直接访问**
- 直接访问链接：https://github.com/settings/profile

### 第三步：进入 Developer settings

1. 在 Settings 页面左侧菜单中，滚动到最底部
2. 点击 **Developer settings**（开发者设置）
3. 或直接访问：https://github.com/settings/developer

### 第四步：选择 Personal access tokens

1. 在 Developer settings 页面左侧菜单中
2. 点击 **Personal access tokens**
3. 选择 **Tokens (classic)**（经典令牌）
4. 或直接访问：https://github.com/settings/tokens

**注意**：GitHub 现在有两种令牌类型：
- **Fine-grained tokens**（细粒度令牌）：新版本，更精细的权限控制
- **Tokens (classic)**（经典令牌）：传统版本，推荐用于 Git 操作

### 第五步：创建新令牌

1. 点击右上角的 **Generate new token** 按钮
2. 选择 **Generate new token (classic)**

### 第六步：配置令牌

#### 1. Note（备注名称）
输入令牌的名称，用于标识这个令牌的用途：
```
例如：Git Push Token for Mac
或：Development Machine Token
```

#### 2. Expiration（过期时间）
选择令牌的有效期：
- **7 days**：7天（适合临时使用）
- **30 days**：30天
- **60 days**：60天
- **90 days**：90天
- **No expiration**：永不过期（不推荐，有安全风险）
- **Custom**：自定义日期

**推荐**：选择 90 days 或 Custom 设置一个合理的过期时间

#### 3. Select scopes（选择权限范围）

对于 Git 推送操作，需要选择以下权限：

**必需权限**：
- ✅ **repo** - 完整的仓库访问权限（推荐）
  - 包括：repo:status, repo_deployment, public_repo, repo:invite, security_events

**可选权限**（根据需要选择）：
- **workflow** - 更新 GitHub Action 工作流
- **write:packages** - 上传包到 GitHub Package Registry
- **read:packages** - 从 GitHub Package Registry 下载包
- **delete:packages** - 删除包
- **gist** - 创建 gists
- **user** - 用户信息读写权限
- **admin:repo_hook** - 仓库钩子管理权限

**最简单配置**：只勾选 **repo** 即可满足 Git 推送需求

### 第七步：生成令牌

1. 滚动到页面底部
2. 点击绿色按钮 **Generate token**

### 第八步：保存令牌

**⚠️ 重要提示**：令牌只显示一次，请立即保存！

1. 复制生成的令牌（类似：`ghp_xxxxxxxxxxxxxxxxxxxx`）
2. 保存到安全的地方，例如：
   - 密码管理器（1Password, LastPass, Bitwarden）
   - 安全的笔记应用
   - 临时文本文件（使用后删除）

**令牌格式示例**：
```
ghp_1234567890abcdefghijklmnopqrstuvwxyz
```

---

## 使用令牌推送代码

### 方式 1：命令行推送（推荐）

```bash
# 执行推送命令
git push origin main

# 系统会提示输入用户名和密码：
# Username for 'https://github.com': wlxx2017
# Password for 'https://wlxx2017@github.com': <粘贴你的令牌>

# 注意：密码处粘贴令牌时不会显示任何字符，这是正常的
```

### 方式 2：在 URL 中嵌入令牌（不推荐，不安全）

```bash
# 格式：https://<token>@github.com/<username>/<repo>.git
git remote set-url origin https://ghp_xxxxx@github.com/wlxx2017/github-dcn.git
git push origin main
```

**⚠️ 警告**：这种方式会将令牌保存在 .git/config 文件中，不安全！

### 方式 3：使用 Git Credential Helper（推荐）

#### macOS - 使用 osxkeychain

```bash
# 配置使用 macOS 钥匙串
git config --global credential.helper osxkeychain

# 推送时输入一次令牌，之后会自动保存
git push origin main
# Username: wlxx2017
# Password: <你的令牌>

# 之后的推送不需要再输入
git push origin main
```

#### Linux - 使用 cache

```bash
# 缓存凭据 1 小时
git config --global credential.helper 'cache --timeout=3600'

# 或缓存 8 小时
git config --global credential.helper 'cache --timeout=28800'

# 推送
git push origin main
```

#### Linux - 使用 store（永久保存，但不安全）

```bash
# 永久保存凭据（明文存储，不推荐）
git config --global credential.helper store

# 推送
git push origin main
```

### 方式 4：使用 Git Credential Manager（推荐）

```bash
# macOS 安装
brew install git-credential-manager

# 配置
git-credential-manager configure

# 推送时会自动打开浏览器进行认证
git push origin main
```

---

## 验证令牌是否工作

### 测试 1：使用 curl 测试

```bash
# 替换 YOUR_TOKEN 为你的令牌
curl -H "Authorization: token YOUR_TOKEN" https://api.github.com/user

# 成功返回示例：
# {
#   "login": "wlxx2017",
#   "id": 123456,
#   ...
# }
```

### 测试 2：使用 Git 测试

```bash
# 尝试列出远程分支
git ls-remote https://wlxx2017:YOUR_TOKEN@github.com/wlxx2017/github-dcn.git

# 或直接推送
git push origin main
```

---

## 管理已创建的令牌

### 查看令牌列表

1. 访问：https://github.com/settings/tokens
2. 可以看到所有已创建的令牌列表
3. 显示信息：
   - 令牌名称
   - 最后使用时间
   - 过期时间

### 编辑令牌

1. 点击令牌名称
2. 可以修改：
   - Note（名称）
   - Expiration（过期时间）
   - Scopes（权限范围）
3. 点击 **Update token** 保存

### 删除令牌

1. 在令牌列表中，点击要删除的令牌
2. 滚动到页面底部
3. 点击红色按钮 **Delete token**
4. 确认删除

**注意**：删除令牌后，使用该令牌的所有应用将无法访问 GitHub

---

## 常见问题解决

### 问题 1：推送时提示 "Authentication failed"

**原因**：令牌权限不足或令牌已过期

**解决**：
1. 检查令牌是否包含 `repo` 权限
2. 检查令牌是否过期
3. 重新生成令牌

### 问题 2：推送时提示 "Permission denied"

**原因**：没有仓库写入权限

**解决**：
1. 确认令牌有 `repo` 权限
2. 确认你对仓库有写入权限
3. 检查仓库名称是否正确

### 问题 3：每次推送都要输入令牌

**原因**：没有配置凭据助手

**解决**：
```bash
# macOS
git config --global credential.helper osxkeychain

# Linux
git config --global credential.helper 'cache --timeout=3600'
```

### 问题 4：令牌泄露了怎么办？

**立即操作**：
1. 访问 https://github.com/settings/tokens
2. 找到泄露的令牌并删除
3. 生成新的令牌
4. 更新所有使用该令牌的配置

### 问题 5：如何查看已保存的凭据？

**macOS**：
```bash
# 查看钥匙串中的 GitHub 凭据
security find-internet-password -s github.com -r "https"

# 或使用钥匙串访问应用
open /Applications/Utilities/Keychain\ Access.app
# 搜索 "github.com"
```

**Linux**：
```bash
# 查看缓存的凭据
git credential-cache get
```

---

## 安全最佳实践

### 1. 定期轮换令牌
- 每 3-6 个月更换一次令牌
- 设置合理的过期时间

### 2. 最小权限原则
- 只授予必需的权限
- 不要勾选不需要的 scope

### 3. 不要分享令牌
- 每个开发者使用自己的令牌
- 不要在团队中共享令牌

### 4. 不要提交到代码仓库
- 不要将令牌写入代码文件
- 不要将令牌提交到 Git
- 使用 `.gitignore` 排除包含令牌的文件

### 5. 使用环境变量
```bash
# 设置环境变量
export GITHUB_TOKEN="ghp_xxxxx"

# 在脚本中使用
git push https://$GITHUB_TOKEN@github.com/user/repo.git
```

### 6. 监控令牌使用
- 定期检查令牌的最后使用时间
- 发现异常立即删除

---

## 完整操作示例

### 示例 1：首次设置并推送

```bash
# 1. 在 GitHub 网站创建令牌（按上述步骤）

# 2. 配置 Git 凭据助手
git config --global credential.helper osxkeychain

# 3. 推送代码
cd /Users/slms26macos/Documents/GitHub/github-dcn
git push origin main

# 4. 输入凭据
# Username for 'https://github.com': wlxx2017
# Password for 'https://wlxx2017@github.com': <粘贴令牌>

# 5. 推送成功
# Enumerating objects: 4, done.
# Counting objects: 100% (4/4), done.
# ...
# To https://github.com/wlxx2017/github-dcn.git
#    ed28c61..b23a0c7  main -> main
```

### 示例 2：使用令牌克隆私有仓库

```bash
# 方式 1：在命令中嵌入令牌
git clone https://wlxx2017:ghp_xxxxx@github.com/wlxx2017/private-repo.git

# 方式 2：推送时输入
git clone https://github.com/wlxx2017/private-repo.git
# Username: wlxx2017
# Password: ghp_xxxxx
```

### 示例 3：为不同项目使用不同令牌

```bash
# 项目 1
cd /path/to/project1
git config credential.helper osxkeychain
git remote set-url origin https://github.com/user1/repo1.git

# 项目 2
cd /path/to/project2
git config credential.helper osxkeychain
git remote set-url origin https://github.com/user2/repo2.git

# 推送时会自动使用对应的凭据
```

---

## 替代方案：SSH 密钥认证

如果您不想使用令牌，可以使用 SSH 密钥：

### 1. 生成 SSH 密钥

```bash
# 生成 ED25519 密钥（推荐）
ssh-keygen -t ed25519 -C "alannetwork1228@gmail.com"

# 或生成 RSA 密钥
ssh-keygen -t rsa -b 4096 -C "alannetwork1228@gmail.com"

# 按提示操作：
# Enter file in which to save the key: <按回车使用默认路径>
# Enter passphrase: <可选，设置密码短语>
# Enter same passphrase again: <再次输入>
```

### 2. 添加公钥到 GitHub

```bash
# 查看公钥
cat ~/.ssh/id_ed25519.pub
# 或
cat ~/.ssh/id_rsa.pub

# 复制公钥内容，然后：
# 1. 访问 https://github.com/settings/keys
# 2. 点击 "New SSH key"
# 3. Title: "Mac Development"
# 4. Key type: "Authentication Key"
# 5. 粘贴公钥内容
# 6. 点击 "Add SSH key"
```

### 3. 更改远程 URL 并推送

```bash
# 更改为 SSH URL
git remote set-url origin git@github.com:wlxx2017/github-dcn.git

# 推送
git push origin main
```

---

## 总结

### 推荐配置流程

1. **创建令牌**：https://github.com/settings/tokens/new
   - Note: "Git Push Token"
   - Expiration: 90 days
   - Scopes: repo

2. **配置凭据助手**：
   ```bash
   git config --global credential.helper osxkeychain
   ```

3. **推送代码**：
   ```bash
   git push origin main
   # Username: wlxx2017
   # Password: <令牌>
   ```

4. **后续推送**：自动使用保存的凭据

### 关键要点

- ✅ 使用 Personal Access Token 替代密码
- ✅ 只授予必需的权限（repo）
- ✅ 设置合理的过期时间
- ✅ 使用凭据助手保存令牌
- ✅ 定期轮换令牌
- ❌ 不要分享或提交令牌
- ❌ 不要使用永不过期的令牌

---

## 参考链接

- [GitHub Token 文档](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [Git Credential Storage](https://git-scm.com/book/en/v2/Git-Tools-Credential-Storage)
- [GitHub Security Best Practices](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure)
