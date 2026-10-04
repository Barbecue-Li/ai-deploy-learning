# Day 01 环境搭建

日期：2026-10-04

## 今日目标

搭好服务器开发环境，写出第一个 C++ 程序，并推送到 GitHub。

## 服务器信息

- 系统：Ubuntu 22.04 x64
- 配置：Intel Xeon Gold，16 核 / 16GB 内存
- 系统盘：30GB（偏小，后面放模型和 Docker 镜像要注意空间，用 `df -h` 查看）
- 带宽：10M（下载大模型较慢，可用 ModelScope 等国内源）

> 笔记中不记录服务器 IP 和密码，避免推到公开仓库后泄露。

## 完成内容

- [x] SSH 登录服务器
- [x] 安装开发工具：g++ 11.4、cmake 3.22、git 2.34、Python 3.10
- [x] 编写并运行第一个 C++ 程序
- [x] 创建 GitHub 仓库 `ai-deploy-learning`，用 SSH 方式推送代码
- [x] 配置本地电脑到服务器的 SSH 免密登录
- [ ] VS Code Remote-SSH 连接（遇到端口转发报错，排查中）

## 操作流程

### 1. 登录服务器

在 Windows PowerShell 中：

```bash
ssh root@服务器IP
```

第一次连接输入 `yes`，输入密码时屏幕不显示，属于正常现象。

### 2. 安装开发工具

```bash
apt update
apt install -y build-essential cmake git python3 python3-pip python3-venv
```

检查是否安装成功：

```bash
g++ --version
cmake --version
git --version
python3 --version
```

### 3. 第一个 C++ 程序

```bash
mkdir ~/ai-deploy-learning
cd ~/ai-deploy-learning
nano hello.cpp
```

```cpp
#include <iostream>
int main() {
    std::cout << "Hello, AI deploy!" << std::endl;
    return 0;
}
```

nano 中 `Ctrl+O` 回车保存，`Ctrl+X` 退出。编译运行：

```bash
g++ hello.cpp -o hello && ./hello
```

### 4. 推送到 GitHub

配置 Git 身份：

```bash
git config --global user.name "GitHub用户名"
git config --global user.email "GitHub邮箱"
```

在服务器上生成 SSH 密钥，并把公钥添加到 GitHub（Settings → SSH and GPG keys）：

```bash
ssh-keygen -t ed25519 -C "邮箱"
cat ~/.ssh/id_ed25519.pub
```

测试连接（看到 `Hi 用户名!` 即成功）：

```bash
ssh -T git@github.com
```

初始化仓库并推送：

```bash
echo "hello" > .gitignore
git init
git add hello.cpp .gitignore
git commit -m "day01: first C++ program"
git branch -M main
git remote add origin git@github.com:Barbecue-Li/ai-deploy-learning.git
git push -u origin main
```

`.gitignore` 里写 `hello`，是为了只上传源代码，不上传编译出来的程序。

### 5. 本地电脑免密登录服务器

在 Windows PowerShell 中，把本地公钥传到服务器（只需输入一次密码）：

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh root@服务器IP "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

编辑 `C:\Users\用户名\.ssh\config`（文件名无后缀）：

```
Host myserver
    HostName 服务器IP
    User root
    IdentityFile ~/.ssh/id_ed25519
```

之后直接用 `ssh myserver` 登录，无需密码。

## 遇到的问题和解决办法

| 问题 | 原因 | 解决办法 |
| --- | --- | --- |
| apt 安装时弹出 “Daemons using outdated libraries” 窗口 | 系统库更新后询问是否重启相关服务，不是报错 | 保持默认，按 `Tab` 选 `<Ok>` 回车 |
| `git push` 时要求输入 Username | 远程地址用的是 HTTPS，GitHub 已不支持密码推送 | 改用 SSH 地址 |
| `git remote add` 报错 `remote origin already exists` | `origin` 已添加过，不能重复 add | 用 `git remote set-url origin 新地址` 修改 |
| 本地 `ssh-keygen` 提示密钥已存在 | 电脑上以前生成过密钥 | 选 `n` 不覆盖，直接用现有密钥 |
| 传公钥时 `Permission denied` | 服务器密码输错 | 重新执行，输入正确密码 |
| SSH config 中有重复条目和 `ForwardAgent yes` | VS Code 之前自动生成 | 只保留 `myserver` 条目，去掉 ForwardAgent |
| VS Code 报错 `Failed to set up dynamic port forwarding` | 通常是服务器禁止了 SSH 端口转发 | 见下方排查步骤 |

VS Code 端口转发问题排查顺序：

1. 服务器上检查：`grep -i forwarding /etc/ssh/sshd_config`，若为 `AllowTcpForwarding no`，改为 `yes` 后执行 `systemctl restart ssh`
2. VS Code 设置中取消勾选 `remote.SSH.enableDynamicForwarding`
3. 服务器上删除残留：`rm -rf ~/.vscode-server`，再重新连接

## 知识点小结

- **SSH 密钥**：公钥放到对方（GitHub / 服务器），私钥留在自己机器上，私钥不能给任何人
- **两套密钥不要混**：服务器上的密钥用于连 GitHub；本地电脑的密钥用于连服务器
- **Git 远程地址**：
  - 第一次设置：`git remote add origin 地址`
  - 修改地址：`git remote set-url origin 地址`
  - 查看地址：`git remote -v`
- **HTTPS vs SSH**：HTTPS 地址以 `https://` 开头，需要 token；SSH 地址以 `git@github.com:` 开头，配好密钥后免密

## 明天计划（Day 02）

- C++：类、构造函数 / 析构函数、RAII
- Linux：文件操作、权限、管道、进程查看（`ps`、`top`）
- 解决 VS Code Remote-SSH 连接问题
- 继续写笔记并推送到仓库