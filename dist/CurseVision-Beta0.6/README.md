# CurseVision Beta0.6

[下载原始发行包](https://github.com/CurSEyy/cursevision/releases/download/distribution-archive/CurseVision-Beta0.6-win-x64.zip)  
SHA-256  `f7953f05e79a77afd6870c96ccfa02f49d8173e6d3835cdebdac041e1699dd22`

## 本版更新

- 增加本机／局域网后端，支持注册、登录、账号资料区分、用户名搜索和双方确认的好友申请。
- 增加蓝、红、橙、黑、紫五种配色、侧栏自定义图片、轻量界面过渡和关闭时收起到系统托盘的选择。
- 沿用 30 秒电源状态采样与插电模式估算。

CurseVision 是工程导论课程的 Windows 寝室用电记录与热水器排程原型。

## Windows 客户端

解压 Windows x64 发布包，再打开 `CurseVision.exe`。请保留整个解压目录。

- 在「设置」切换蓝色、红色、橙色、黑色、紫色主题，或导入侧栏图片。支持 PNG、JPEG、WebP、BMP 和 GIF，GIF 使用第一帧。
- 右上角关闭按钮默认询问收起到系统托盘还是退出，可以记住选择。托盘图标支持恢复窗口和退出软件。采集已开启时，收起后继续采样。
- Beta0.5 的记录仍可离线使用。登录后打开当前账号在这台电脑上的独立用电资料，退出登录后返回离线资料。如果启用了旧版的 Windows 登录启动并把新版放在新目录，可在新版设置里关闭并重新开启启动选项。

## 账户后端

1. **同一台电脑试用**。先运行发布包中的 `backend/CurseVisionServer.exe`，保持窗口开启。在客户端打开「账户与好友」，用默认地址 `http://127.0.0.1:8765` 注册或登录。
2. **舍友一起使用**。准备一台后端电脑，以及其他电脑可以访问的局域网 IPv4 地址和 TCP 8765 端口。后端电脑运行 `backend/Start-LAN.cmd`。所有成员把客户端地址改为 `http://后端电脑IPv4:8765`。使用 `ipconfig` 可查看主机的 IPv4 地址。如果校园网隔离了设备，换到可互通的局域网或使用已经部署的 HTTPS 后端。若防火墙拦截，需在后端电脑允许可信专用网络访问该端口。
3. 注册两个不同用户名，使用不同账号登录。A 搜索 B 并发起申请。B 在「好友与申请」刷新、选择该申请、点击「同意」，双方即成为好友。搜索支持完整用户名或前缀。

同机演示无需云主机、域名、数据库账号或 API Key。局域网 HTTP 适合可信演示环境。跨公网时准备部署主机与能让成员访问的 HTTPS 地址。

后端在首次运行时自动创建数据目录。Windows 默认使用 `%LOCALAPPDATA%\CurseVisionServer`，可通过 `--data-dir` 指定位置。备份时分别保管 `accounts.enc` 与配套的 `server.key`。账户和好友关系加密保存，密码以加盐哈希保存。后端重启后重新登录。

## 用电与排程演示

1. 在「用电概览」选择开启采集。软件每 30 秒读取本机电源状态。插电时按在设置里选中的模式功率估算用电；只靠电池供电时，电脑在插座一侧记为 `0 W`。Windows 能提供有效电池读数时，另行显示电池充放电瓦数。
2. 在「成员与共享」导出近 7 天本机记录，再在寝室长电脑上导入。重复导入会跳过已有时刻。可从选中的成员转交寝室长。
3. 在「热水器排程」点「试算示例」查看开启时间、档位和预计电量。也可以录入参数和真实预约。软件给出建议，寝室长可另行登记已经结束的实际加热时段。

离线用电数据保存在 `%LOCALAPPDATA%\CurseVision\records.sqlite3`，各账号的本机用电数据放在同目录的 `profiles` 子目录。后端负责账户和好友，寝室用电采用主动导入和导出。

## 阅读源码

安装 Python 3.13 Windows 版并保留 Tcl/Tk 组件。用 `python -m pip install -r requirements.txt` 安装依赖，通过 `python server.py` 启动后端，通过 `python app.py` 启动客户端。后端也可在安装 Python 和 cryptography 的 Linux 主机上启动。

| 文件 | 内容 |
| --- | --- |
| `app.py` | 界面、采样定时器、账户交互 |
| `appearance.py` | 五种主题、图片导入和托盘 |
| `account_client.py` | 客户端账户接口 |
| `server.py` | 注册、登录、好友确认和加密存储 |
| `records.py` | SQLite、耗电量和寝室文件交换 |
| `windows_power.py` | Windows 电源接口和登录启动 |
| `heating.py` | 三档热水器排程 |

将 `CurseVision.ico` 放到客户端 EXE 旁边可更换窗口及托盘图标。源码运行时放到 `app.py` 旁边。
