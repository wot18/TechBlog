

## 一、 数据迁移核心思路

WorkBuddy 在运行过程中会在 C 盘用户目录下产生大量数据（如`.workbuddy` 文件夹和  `WorkBuddy` 文件夹），导致 C 盘空间紧张。

**核心解决策略：使用“目录联接” (Directory Junction)**

1. **物理转移**：将真实的数据文件夹移动到空间充裕的磁盘，如 D 盘（例如 `D:\WorkBuddyData\` 目录下）。
    
2. **逻辑占位**：在 C 盘原数据路径处，创建一个指向 D 盘新路径的“目录联接 (Junction)”。
    
3. **无感运行**：由于 Windows 系统的目录联接特性，WorkBuddy 应用程序在读取 C 盘路径时，系统会自动将其透明重定向到 D 盘。应用本身毫无感知，从而安全实现空间释放。
    

## 二、 核心操作步骤与命令

请在**管理员终端**（CMD）中依次执行以下操作。注意：以下命令以系统用户名 `AiKing` 和目标 D 盘存放路径 `D:\WorkBuddyData` 为例，请在实际操作时替换为您自己的用户名和路径。

### 第 0 步：彻底退出程序（关键）

必须确保 WorkBuddy 已彻底退出，任务管理器中无相关进程。否则后续重命名时会因为文件被占用而报错“访问被拒绝”。

### 第 1 步：复制数据到 D 盘

使用 `robocopy` 命令将 C 盘的数据完整复制到 D 盘（`/E` 参数表示复制所有子目录，包括空目录）。

```
robocopy "C:\Users\AiKing\.workbuddy" "D:\WorkBuddyData\workbuddy" /MIR /R:2 /W:3 /XJ 
robocopy "C:\Users\AiKing\WorkBuddy" "D:\WorkBuddyData\WorkBuddy" /MIR /R:2 /W:3 /XJ 
```

### 第 2 步：重命名原目录作为备份

使用 `rename` 命令将 C 盘原有的数据文件夹添加 `.bak` 后缀。这一步非常重要，如果迁移失败可以随时将名字改回来进行**回滚恢复**。

```
rename "C:\Users\AiKing\.workbuddy" ".workbuddy.bak"
rename "C:\Users\AiKing\WorkBuddy" "WorkBuddy.bak"
```

_(注：如果使用的是 PowerShell 终端，请将 `rename` 替换为 `Rename-Item`)_

### 第 3 步：创建目录联接

使用 `mklink /J` 命令在 C 盘创建软链接（Junction 目录联接），将其透明指向 D 盘的新位置。

```
mklink /J "C:\Users\AiKing\.workbuddy" "D:\WorkBuddyData\workbuddy"
mklink /J "C:\Users\AiKing\WorkBuddy" "D:\WorkBuddyData\WorkBuddy"
```

### 第 4 步：功能验证

1. 重新启动 WorkBuddy 并登录。
    
2. 检查历史会话列表是否完整。
    
3. 提问“你还记得我是谁/你是谁”以确认记忆/身份文件加载正常。
    
4. 检查设置与插件是否正常加载。
    
5. 在资源管理器中查看 C 盘原路径属性，确认已显示为“联接”。
    

## 三、 注意事项与排错

- **权限要求**：必须右键“开始”菜单，选择“终端(管理员)”或“命令提示符(管理员)”来执行上述命令。
    
- **报错“访问被拒绝”**：如果在执行 `rename`（第2步）命令时提示访问被拒绝，说明 WorkBuddy 程序没有退干净，目录被锁住了。解决方法是回到第 0 步，在任务管理器中强制结束所有 WorkBuddy 进程后再试。
    
- **防丢机制**：就算操作失败也不用慌张，因为此时真实数据已在 D 盘有一份拷贝，且 C 盘原数据只是被重命名为 `.bak`，随时可以通过删除创建的联接并去掉原文件的 `.bak` 后缀来撤销恢复。