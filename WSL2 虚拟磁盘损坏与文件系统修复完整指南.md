
这份文档记录了 WSL2（Windows Subsystem for Linux）在遭遇意外断电、死机或被强杀后，导致虚拟磁盘（VHDX）损坏、系统进入只读模式，并伴随 `Structure needs cleaning` 报错的完整排查与终极修复流程。

---

## 🛑 故障现象回顾

在本次修复过程中，系统依次出现了以下典型的连环报错：

1. **WSL 无法启动，报灾难性故障**
   > `错误代码: Wsl/Service/CreateInstance/E_UNEXPECTED`
   > `[已退出进程，代码为 4294967295 (0xffffffff)]`
2. **强制重启启动后，系统进入只读模式**
   > `wsl: An error occurred mounting the distribution disk, it was mounted read-only as a fallback.`
3. **尝试在 Windows 侧挂载磁盘修复时，遭遇死锁被拒绝访问**
   > `无法将磁盘“...\ext4.vhdx”附加到 WSL2: 另一个程序正在使用此文件，进程无法访问。`
   > `错误代码: Wsl/Service/CreateInstance/MountDisk/HCS/ERROR_SHARING_VIOLATION`
4. **进入 WSL 内部查看文件时，发现文件系统结构损坏**
   > `ls: cannot access 'Programs': Structure needs cleaning`

---

## 🛠️ 终极修复方案（绕过 Windows 占用的内部修复法）

当 Windows 侧频繁提示 `ERROR_SHARING_VIOLATION` 且无法解除 `ext4.vhdx` 磁盘占用时，最有效的方法是**直接利用 WSL 的“只读后备模式（read-only fallback）”进入系统内部进行底层修复**。

### 第一步：以 Root 权限直接进入受损的 WSL
打开 Windows 的 **PowerShell（管理员）**，输入以下命令强制以最高权限进入报错的 Linux 分发版：
```bash
# 将 Ubuntu 或者你的分发版名字替换掉这里的 <DistributionName>
wsl -d <DistributionName> -u root
```
*(注：如果此时终端弹出 `read-only as a fallback` 警告，请忽略并按回车，你会看到命令提示符变成 `#`，说明已成功进入内部。)*

### 第二步：找出真实的损坏磁盘代号
由于底层虚拟化映射，直接使用 `findmnt` 可能会报错 `none`。我们需要手动找出挂载的主系统盘代号：
```bash
mount | grep ext4
```
**输出示例：**
> `/dev/sdc on / type ext4 (ro,relatime,discard,errors=remount-ro,data=ordered)`

提取行首的设备路径。通常是 `/dev/sdc`、`/dev/sdb` 或 `/dev/sdd`。请记住这个路径。

### 第三步：执行强制修复（e2fsck）
针对刚刚找到的磁盘代号，执行 ext4 文件系统的检查与自动修复工具：
```bash
# 将 /dev/sdc 替换为你实际查到的代号
e2fsck -f -y /dev/sdc
```
**参数解释：**
* `-f`：强制检查（Force check），即使系统标记为干净也强行扫描。
* `-y`：全部自动回答 Yes，无需人工干预确认每一个节点的修复。

此时屏幕会滚动输出大量的修复日志（包括清理孤立的 inode、修复 block 错误等）。等待其运行完毕并重现 `#` 提示符。

### 第四步：彻底重启 WSL
修复成功后，当前的 Linux 环境依然处于之前的状态，需要彻底重启服务。
依次输入以下命令：
```bash
# 1. 退出 Linux 内部环境
exit

# 2. 回到 Windows PowerShell 后，彻底关闭所有 WSL 进程
wsl --shutdown
```

完成上述操作后，再次从开始菜单或终端正常打开你的 Ubuntu，`Structure needs cleaning` 的报错将彻底消失，系统恢复正常的读写状态。

---

## 💡 附录：其他相关的排查指令

如果故障仅停留在**无法启动**（E_UNEXPECTED）而未涉及磁盘损坏，可能是由于 Windows 网络协议栈冲突或服务僵死导致的，可以尝试以下修复指令：

**1. 重置 Winsock（网络冲突）**
```cmd
netsh winsock reset
# 执行后必须重启电脑
```

**2. 重启底层虚拟计算服务（解除一般的死锁占用）**
```cmd
wsl --shutdown
net stop vmcompute
net start vmcompute
```

