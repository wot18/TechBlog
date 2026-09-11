

这份文档记录了将本地项目（如 Python 番茄钟项目）首次推送到 GitHub 的完整流程，以及常见报错的解决办法和日常使用技巧。

## 第一部分：首次上传本地项目到 GitHub

### 1. 准备工作：创建 `.gitignore` 文件（非常重要）

为了防止将本地生成的缓存文件、IDE 配置文件或私密数据上传到公开仓库，必须在项目根目录创建一个名为 `.gitignore` 的文件。

针对 Python 项目，推荐配置如下：

```
# Python 编译缓存
__pycache__/
*.py[cod]
*$py.class

# IDE 和工具缓存
.vscode/
.ruff_cache/
.idea/

# 本地生成的私人数据或配置（视情况而定）
pomodoro_data.json
pomodoro_config.json
```

### 2. 初始化本地 Git 仓库

在终端中进入你的项目根目录，依次执行以下命令：

```
# 1. 初始化 Git 仓库
git init

# 2. 将所有未被 .gitignore 忽略的文件添加到暂存区
git add .

# 3. 提交这些文件到本地仓库，并附带说明信息
git commit -m "Initial commit: 首次提交项目基础代码"
```

### 3. 关联 GitHub 远程仓库并推送代码

在 GitHub 网页端创建一个**空仓库**（⚠️ 建议创建时**不要**勾选自动生成 README 或 License，以免引起冲突）。然后执行以下命令：

```
# 1. 将默认分支重命名为 main（现代 GitHub 的标准）
git branch -M main

# 2. 关联本地仓库与 GitHub 远程仓库（请替换为你的真实链接）
git remote add origin [https://github.com/你的用户名/你的项目名.git](https://github.com/你的用户名/你的项目名.git)

# 3. 首次推送代码，并建立本地与远程 main 分支的追踪关系
git push -u origin main
```

## 第二部分：常见报错与解决办法

### 报错 1：`Updates were rejected because the remote contains work that you do not have locally.`

- **原因**：通常是因为在 GitHub 创建仓库时，勾选了自动生成 `License` 或 `README` 文件。导致 GitHub 上的仓库有内容，而本地代码不知道这些内容，Git 出于安全机制拒绝覆盖。
    
- **解决办法（两种任选其一）**：
    
    - **方法 A（推荐：强行合并）**：先将远端文件拉取下来合并，再重新推送。
        
        ```
        git pull origin main --allow-unrelated-histories
        git push -u origin main
        ```
        
        _(如果在 pull 时弹出 Vim 编辑器界面，直接按 `Esc`，输入 `:wq` 然后按回车即可。)_
        
    - **方法 B（粗暴覆盖）**：如果确认远端只有无用的初始文件，可以直接强制覆盖（远端内容会丢失）。
        
        ```
        git push -f origin main
        ```
        

### 报错 2：`The current branch main has no upstream branch.`

- **原因**：本地的 `main` 分支不知道要把代码推送到远端的哪一个分支（即没有建立 Upstream 追踪关系）。
    
- **解决办法（两种任选其一）**：
    
    - **方法 A（单次解决）**：按照提示，在推送时指定目标并建立绑定。
        
        ```
        git push --set-upstream origin main
        ```
        
    - **方法 B（一劳永逸的全局设置）**：设置 Git，让它以后遇到没有绑定远端的分支时，自动在远端创建一个同名分支。**强烈推荐执行一次此命令**：
        
        ```
        git config --global push.autoSetupRemote true
        ```
        
        设置之后，以后直接输入 `git push` 即可，再也不会报这个错。
        

## 第三部分：🏆 日常开发提交“三步曲”

当你首次把代码推送到 GitHub 并解决好报错后，以后的日常开发就会变得非常简单。 每次你修改了代码、增加了新功能、或修复了 Bug，只需要在终端执行这简单的三步：

```
# 第一步：把所有修改过的文件添加到暂存区
git add .

# 第二步：把修改提交到本地仓库，写上这次改了什么
git commit -m "描述这次修改的内容，比如：修复了倒计时卡顿的bug"

# 第三步：把本地的新版本推送到 GitHub 保存
git push
```

**💡 提示**：经常进行 `commit` 和 `push` 是一个好习惯，它就像是你在玩游戏时的“存档”，就算代码改坏了，也能随时回滚到之前的正常状态。