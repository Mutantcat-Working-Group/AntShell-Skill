<div align="center">
<img src="https://file001.mutantcat.org/view.php/3006811ff7773292f188ceedf52ab4e2.png" style="width:100px;" width="100"/>
<h2>蚂蚁终端-Skill</h2>
</div>


### 一、产品概述

- 一款面向开发者与运维的现代终端工具，覆盖 SSH、FTP、本地终端三大核心场景
- 发行方：由异猫工作群（mutantcat.org）发行，GitHub：<https://github.com/Mutantcat-Working-Group>
- 支持拖拽式上传文件/文件夹、支持在线编辑文本文件、递归下载文件夹、快速删除
- 集成 AI 问答能力，命令行问题、系统排错、脚本生成等可即时获得建议
- 集成 AI 直接操作命令行能力，AI直接帮你操作终端，发送指令
- 支持 MCP（Model Context Protocol）开关，可按需启用工具链调用
- 支持一键安装适配 MCP 的Skill，实现全过程 AI运维 外部调用
- MCP 操作软件全部功能与配置、支持同步操作软件内容、创建连接、操作命令行
- 标签化工作台，多开支持多窗口并行操作

### 二、软件界面

- 连接管理
- ![welcome.png](https://mutantcat.s3.bitiful.net/antshell/91a7ab832045e5a23a39ba8fb99efd78.png?no-wait=on)
- SSH 工作台
- ![ssh.png](https://mutantcat.s3.bitiful.net/antshell/64984a6d3ff43825da2b4c4ee5312002.png?no-wait=on)
- FTP 工作台
- ![ftp.png](https://mutantcat.s3.bitiful.net/antshell/8c708567fe1d56123a6eac95ba2bbb27.png?no-wait=on)
- AI 问答与 MCP 工具链
- ![ai1.png](https://mutantcat.s3.bitiful.net/antshell/c312f5f808784eff76a1fd18573fa0bb.png?no-wait=on)
- ![ai2.png](https://mutantcat.s3.bitiful.net/antshell/1c6c161656e5c1c984bd003d06a123b8.png?no-wait=on)
- 本地终端
- ![local.png](https://mutantcat.s3.bitiful.net/antshell/12427afa6eba10253401fc17ad857c15.png?no-wait=on)

### 三、功能说明

#### SSH 工作台

- 标签页式多会话管理，每个会话独立保留历史
- 支持密钥登录与密码登录，密钥可绑定到具体连接
- 支持文件拖拽上传、递归下载、文件移动、权限编辑
- 支持文本文件在线查看、编辑、保存

#### FTP 工作台

- 单面板远端文件浏览、下载（现代化）
- 支持拖拽上传、重命名、删除
- 支持断点续传与大文件分片传输
- 支持 SFTP（基于 SSH）与显式 FTPS
- 支持传输队列，随时取消传输

#### 本地终端

- 内置跨平台、多平台 Shell（PowerShell / CMD / bash / zsh 等）
- 可复用 SSH 工作台的标签、字体、配色与快捷键
- 支持 Shell 主题切换与历史命令搜索
- 支持 AI 工具：打开终端时自动打开 AI 工具（Claude Code、OpenCode等）
- 支持工具栏：通过工具链打开的窗口，发送指令前自动拼命工具链
- 
#### AI 问答

- 在终端内随时唤起 AI 助手
- 支持上下文范围（当前会话、最近N行、自定义文本）
- 支持将 AI 建议直接以命令片段形式插入到终端

#### MCP 工具链

- 开关式启用 MCP（Model Context Protocol）能力
- 支持常见 MCP 服务：文件操作、Shell 命令、程序配置修改
- 工具调用前可在 UI 预览与审批

### 四、激活说明

- 未激活状态下：
    - 可保存的连接信息数 ≤ 20
    - SSH 工作台、FTP 工作台最大打开窗口数 ≤ 2
    - AI 问答限 15 轮
    - MCP 默认关闭
    - 工具链可存储 3 条
- 激活专业版后解锁：
    - 可保存 20 个以上的连接信息
    - SSH 工作台、FTP 工作台无限并发窗口
    - AI 问答支持无限轮
    - 开放 MCP 开关
    - 允许配置无限个工具链
- 激活方式：
    - 在线激活：在软件内点击在线激活，浏览器选择席位后自动完成在线激活并绑定当前设备
    - 离线激活：在软件激活窗口获取机器码后到主站用户中心提交，生成激活码
注意：激活是一机一码

### 五、依赖与运行

- Windows 10+、macOS 11+、主流 Linux 发行版（Ubuntu / Debian / Fedora / Arch 等）
- 首次启动会自动检测终端环境与必要依赖

