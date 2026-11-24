# 北交校园网助手 0.1.1 — Agent 接手说明

[下载原始发行包](https://github.com/CurSEyy/cursevision/releases/download/distribution-archive/BJTUCampusWifi-0.1.1-complete-handoff.1.zip)  
SHA-256  `661aed5c13e7101202d73fd079789ec87a61f43116263aa5e208999ee98f0cb5`

## 本版更新

- 包含独立版校园网助手可执行程序、用法、源码、构建脚本及 SHA-256 清单。
- 该交接包沿用上游 0.1.1 的源码和可执行程序，新增接手说明与文件清单。CurseVision 从 alpha0.9 开始内置该功能。

本包交付当前已发布的 Windows 电脑版测试版，供用户运行及其他 agent 继续维护。
整理日期为 2026-11-23。

## 从这里开始

普通用户打开 `program/BJTUCampusWifi.exe`。
开发或接手维护时，从 `source/desktop_app.py` 阅读，再看
`source/campus_wifi.py` 和 `source/campus_connection.py`。
`source/README.md` 面向使用者，本文件补充实现细节和测试边界。

目录结构如下。

```text
README.md                    本接手说明
MANIFEST.json                包内其他文件的 SHA-256、大小和来源
program/
  BJTUCampusWifi.exe         已发布的 Windows x64 单文件程序
  使用说明.txt               使用步骤
  THIRD_PARTY_LICENSES/      第三方组件授权文件
source/                      已发布版本的完整源码与构建脚本
```

项目地址为 https://github.com/michaelpotter8894-cyber/bjtu-campus-wifi 。
版本页为 https://github.com/michaelpotter8894-cyber/bjtu-campus-wifi/releases/tag/v0.1.1 。
对应的源码提交为 `13412037da07f6315dda0e49d33ae464745c0223`。
本次整理复用该版本的 exe 和源码，交接包新增本说明与文件清单。

## 目标和运行条件

在 Windows 登录、睡眠或休眠恢复、校园 Wi-Fi 重连时，自动完成北交校园网网页认证。
支持 Windows 10/11 x64、北京校区的 `web.wlan.bjtu`。
用户需要拥有可用的校园网上网账号，并先让 Windows 连上校园 Wi-Fi。
程序优先使用 Edge，在浏览器启动失败时回退到 Chrome。
两者至少需要安装一个，用户平时可以使用其他浏览器浏览网页。

程序依赖 Windows 凭据管理器、任务计划程序、Windows PowerShell、
Wi-Fi 状态查询、Tkinter 和 Playwright。
发布 exe 已携带 Python、Playwright 驱动及相关依赖。

## 模块职责

| 文件 | 职责 |
| --- | --- |
| desktop_app.py | GUI 入口、后台模式、日志、安装与卸载、并发锁 |
| campus_wifi.py | Wi-Fi 检查、凭据读写、认证页导航、表单识别与提交 |
| campus_connection.py | 获取校园网卡 IPv4，建立限于认证用途的本地连接通道 |
| task_setup.ps1 | 注册或移除登录、唤醒和 Wi-Fi 重连触发的计划任务 |
| shortcut.ps1 | 创建开始菜单快捷方式 |
| uninstall.ps1 | 等待当前进程退出后清理专用安装目录 |
| build.ps1 | PyInstaller 打包入口，嵌入三个 PowerShell 资源 |
| requirements-build.txt | 顶层构建依赖版本 |
| set_account.pyw | 原始脚本的独立凭据录入窗口 |
| 两个 cmd 文件 | 原始开发机启动方式，依赖已有的开发虚拟环境路径 |

## 认证流程和入口差异

`desktop_app.py` 无参数时显示窗口。
`--unattended` 使用已保存的凭据在后台认证，任务计划程序调用此模式。
`--check` 仅打开认证页、检查状态或识别表单，可以用它检查网络路径。
窗口的「立即认证」调用相同的认证流程。

核心流程为检查 SSID、等待 Wi-Fi、取得网卡地址、启动认证连接通道、
启动后台浏览器、识别已有登录状态或登录表单、读取凭据、提交一次、检查返回页面。
后台模式等待 Wi-Fi 的时间上限约一分钟，其他步骤还有各自的超时。

登录页真实可见控件的名称为 `DDDDD`（账号）、`upass`（密码）、
`0MKKey`（按钮）。页面还包含同名隐藏控件，定位器使用可见性筛选。
成功判断主要依赖校方 HTTPS 域名和页面标题，例如「注销页」「登录成功」。
这个判断确认的是认证页状态，后续维护可补充独立的外网连通性验证。

`campus_wifi.py` 也可以独立运行，支持 `--configure`、`--check`、
`--unattended`。它与桌面入口有区别，独立入口会覆盖当前日志，
桌面入口使用滚动日志。桌面入口还通过 `msvcrt` 文件锁限制认证并发，
独立脚本入口没有这层文件锁。

## 网络实现的原因与边界

曾遇到 Clash 的虚拟 DNS 地址及默认网络路径导致校方 HTTPS 访问失败。
开发机上绑定校园 Wi-Fi 网卡直接访问后，证书验证和页面访问成功。
因此 `campus_connection.py` 用源地址绑定校园网卡并连接固定目标
`10.10.42.3`。浏览器访问的主机名仍为 `login.bjtu.edu.cn`，
TLS 仍由浏览器验证。

本地服务只监听 `127.0.0.1` 的随机端口，在本次运行退出时关闭。
CONNECT 当前允许校方域名的 443 和 802 端口。
HTTP GET 当前接受校方域名和微软连通性检查域名，
这两类请求在本连接通道内均发往校内固定服务器的 80 端口，
用于触发认证重定向。这里的微软检查网址不能当作独立的外网可达性测试。
其他目的地被本地服务拒绝。

固定 IP、端口、网页字段、标题和本地化的网卡输出解析均可能需要随学校环境调整。
其他校区、多网卡、多 IPv4 地址及不同语言的 Windows 尚需验证。

## 本机状态及安装副作用

- 安装目录为 `%USERPROFILE%\AppData\Local\BJTUCampusWifiApp`。
- 计划任务名为 `BJTU Campus Wi-Fi`，安装会替换同名旧任务。
- 任务以当前用户的交互式登录身份、普通权限运行。
- 登录 Windows 时立即触发，另外监听 System 的唤醒事件和校园 Wi-Fi 的重连事件。
- 任务允许电池供电运行，执行上限四分钟，失败后每分钟重试、最多三次。
- 多实例策略为忽略新实例。
- 开始菜单入口为 `BJTU Campus Wi-Fi.lnk`。
- 凭据服务名为 `BJTU Campus Wi-Fi`，条目名为 `portal-account`。
- 凭据内容为 JSON，通过 `WinVaultKeyring` 写入 Windows 凭据管理器。
- 日志和认证锁位于 `%USERPROFILE%\.bjtu-campus-wifi`。
- 桌面入口日志最多约 200 KB，保留一个滚动备份。

用户首次输入自己的账号密码。旧脚本和桌面版本共享凭据与任务名。
账号、密码和认证 URL 的查询参数不应进入日志、源码提交或反馈材料。
接手排查时先检查日志中的状态和错误类型，再检查任务运行记录、SSID、
网卡地址以及浏览器启动结果。

安装失败可能留下已复制的 exe。已安装程序运行中覆盖升级可能失败。
卸载通过独立 PowerShell 进程等待应用退出后清理安装目录，
错误输出目前被重定向，维护时可改进清理结果反馈。
卸载可选择保留凭据，下载目录中的副本及日志需要用户自行管理。

## 从源码运行与构建

在 Windows PowerShell 终端进入 `source` 目录，使用 Python 3.13 x64。

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-build.txt
.\.venv\Scripts\python.exe desktop_app.py
.\.venv\Scripts\python.exe desktop_app.py --check
.\build.ps1 -Python .\.venv\Scripts\python.exe
```

构建输出为 `source/release/BJTUCampusWifi.exe`。
也可以用 `-ReleaseDirectory release/0.1.1` 指定输出目录。
GUI 的安装操作要求使用打包后的 exe。
构建脚本生成 exe，完整发布压缩包还需配入使用说明与第三方授权目录。

顶层依赖为 Playwright 1.65.1、keyring 25.7.0、PyInstaller 6.22.3。
传递依赖未全部锁定，重新构建的二进制字节可能不同。
构建时需要保留三个 PowerShell 资源和 Playwright 的打包 hook。
原始 cmd 文件面向开发机已有虚拟环境，通常应直接使用上述命令或 exe。

## 已有验证和待验证部分

此前已在开发机完成以下检查。

- 原始脚本真实登录成功，校方页面确认认证成功，随后检查过外网访问。
- Chrome 与 Edge 分别打开校方认证页并识别登录状态。
- Edge 启动失败时回退 Chrome，以及两个浏览器均无法启动的错误路径。
- 打包后的 0.1.1 exe 通过 `--check`，日志确认实际使用了 Edge。
- 合成凭据的保存与读取。
- 安装文件复制、快捷方式创建、隔离测试计划任务运行和卸载清理。
- 本地连接通道拒绝其他 HTTPS 目标。
- 发布文件上传完成，远端文件大小及可用的 SHA-256 摘要校验通过。

本次交接打包执行文件完整性核对，既有测试记录不等于在所有设备上验证。
新 exe 在未认证状态、其他同学的电脑、真实重启和休眠恢复场景仍需更多实测。
当前程序未签名，发布状态为测试版。
仓库目前未给项目自有代码指定开源许可证，第三方授权随包附带。
未来公开授权范围由项目所有者确定。

## 接手后的验证顺序

1. 查看 `MANIFEST.json` 并核对文件完整性，阅读三个主要 Python 模块。
2. 在 Windows 上运行 `--check`，确认 Wi-Fi、网卡地址及校方页面可达。
3. 在用户允许短暂断网时测试未认证状态下的真实登录，观察日志和认证页。
4. 涉及安装或任务修改的测试可用 `task_setup.ps1` 的 `-TaskName` 参数创建独立测试任务。
5. 按改动范围测试凭据读取、Chrome/Edge 回退、网络错误和卸载清理。

本包的代码版本与已发布的 v0.1.1 对齐。
后续修复应更新版本说明、重建 exe，并重新生成发布包的校验信息。
