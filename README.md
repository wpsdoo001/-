# 快捷中心（fnshortcut）for 飞牛 fnOS

在飞牛桌面里管理自己的「网址 / IP 快捷方式」：手动输入 IP 或网址 → 自动抓图标或生成字母图标 → 点击就在飞牛桌面窗口内打开（内置 iframe 窗口，也支持新标签页、浏览器直接跳转）。

- 应用名：`fnshortcut`
- 桌面入口：快捷中心（`fnshortcut.main`，type=`iframe`，在 fnOS 桌面窗口内打开）
- 服务端口：`8899`（应用内可改，见下文）
- 平台：`all`（x86 / ARM 通用，只依赖 Python 3）
- 安装包：`fnshortcut.fpk`

## 一、功能

- **快捷方式管理**：名称、地址（IP、`192.168.1.1:8080`、`https://xxx` 都能识别，缺协议自动补 `http://`）、备注、打开方式、图标、颜色。
- **图标**：自动抓取网站 favicon（先解析首页 `<link rel="icon">`，退回 `/favicon.ico`）；失败则按名称生成渐变字母图标；也可上传本地图片或直接用文字图标。
- **桌面窗口内打开（核心）**：飞牛桌面点图标 → 打开「快捷中心」→ 点卡片 → 在应用内的多标签窗口里打开目标页面。
  - `内嵌窗口`：直接把目标地址放进 iframe。
  - `兼容模式`：当目标站点禁止被嵌入（`X-Frame-Options` / CSP）时，由内置服务端代理转发，去掉禁用嵌入的响应头并重写页面链接，多数内网设备（路由器、下载器、影音库、Docker 面板）都能正常显示。
  - `浏览器标签页`：直接新开标签页，适合必须登录或对嵌入极敏感的站点。
- **卡片视图**：网格 / 列表两种布局、三档卡片大小、在线状态绿点（并发探测端口）、深/浅色主题、搜索（快捷键 ⌘K / Ctrl+K）、拖不动的纯前端零依赖实现。
- **数据可带走**：设置里支持导出 / 导入 JSON 备份，卸载重装不丢配置（前提是同一数据目录）。

## 二、安装

### 方式 1：应用中心「手动安装」（推荐）

1. 把 `fnshortcut.fpk` 下载到电脑或 NAS 上；
2. 打开飞牛桌面 → **应用中心** → 左下角 **手动安装** → 选择该 fpk（支持「从电脑上传」和「从 NAS 添加」）；
3. 按提示完成安装，等应用状态变成「运行中」；
4. 回到桌面，点 **快捷中心** 图标即可使用。

> 说明：本包为**未签名包**（`fnpack build --nosign`）。飞牛的开发者签名证书需在飞牛应用开放平台申请，本地自签证书会被 `fnpack` 拒绝（`Invalid certificate.`），因此这里没有签名。
> 多数版本的「手动安装」允许在提示来源/签名风险后继续安装；如果系统直接拒绝该包，请用下面的方式 2。

### 方式 2：SSH 命令行安装（手动安装被拒时）

NAS 开启 SSH 后，用管理员账号登录：

```bash
# 直接把 fpk 装进去
sudo appcenter-cli install-fpk /path/to/fnshortcut.fpk

# 或者把源码目录（fnshortcut/）传到 NAS，本地打包并安装
cd /path/to/fnshortcut
sudo appcenter-cli install-local
```

安装完成后：

```bash
sudo appcenter-cli list          # 确认 fnshortcut 已安装
sudo appcenter-cli start fnshortcut
```

### 方式 3：不装应用，直接跑（临时体验）

```bash
cd fnshortcut/app
TRIM_PKGVAR=/tmp/fnsc python3 server/server.py    # 默认监听 8899
# 浏览器打开 http://NAS_IP:8899
```

## 三、使用

1. 打开「快捷中心」，点右上角 **新建**；
2. 填名称 + 地址（例如 `192.168.31.1`、`http://nas:5244`、`https://www.baidu.com`）；
3. 选打开方式：一般先用「内嵌窗口」，页面白屏/拒绝连接就改成「兼容模式」；
4. 保存后卡片出现，点卡片就在应用窗口内打开；卡片右下角 `⋯` 还有「浏览器打开 / 编辑 / 重抓图标 / 复制地址 / 删除」。

HTTPS 站点被嵌在 `http://NAS_IP:8899` 页面里时，个别浏览器会拦截混合内容；遇到这种情况用「浏览器标签页」方式打开即可。

## 四、端口与数据

- 服务端口默认 `8899`，被占用时可在 `manifest` 的 `service_port`、`app/ui/config` 的 `port` 两处改一致后重新打包；
- 运行数据在应用数据目录（`TRIM_PKGVAR`）下：
  - `data/shortcuts.json`：快捷方式与设置
  - `data/icons/`：上传/抓取的图标
  - `info.log`：运行日志
- 程序只依赖 Python 3 标准库（`http.server`），不需要 pip 安装任何包，也不需要 root 权限运行（`config/privilege` 里 `run-as: package`）。

## 五、目录结构

```
fnshortcut/
├── manifest                 # 应用元信息（appname/version/service_port/入口…）
├── ICON.PNG / ICON_256.PNG  # 应用中心与桌面的 64 / 256 图标
├── app/
│   ├── ui/config            # 桌面入口定义（iframe + port 8899）
│   ├── ui/images/           # 入口图标 icon_64.png / icon_256.png
│   ├── server/server.py     # 后端：静态页 + 快捷方式 API + favicon 抓取 + 兼容代理
│   └── web/                 # 前端（原生 HTML/CSS/JS，无构建）
├── cmd/main                 # start / stop / status
├── cmd/install_* 等         # 生命周期脚本
├── config/privilege         # run-as: package
└── config/resource          # 不额外声明共享目录
```

## 六、自己重新打包

```bash
# 准备 fnpack（Linux amd64 版即可）
curl -O https://static2.fnnas.com/fnpack/fnpack-1.3.3-linux-amd64 && chmod +x fnpack-1.3.3-linux-amd64

# 未签名包（本地调试用）
./fnpack-1.3.3-linux-amd64 build -d fnshortcut --nosign

# 已签名包（需要飞牛开放平台签发的开发者证书）
./fnpack-1.3.3-linux-amd64 cert import -key developer.key -crt developer.crt
./fnpack-1.3.3-linux-amd64 build -d fnshortcut
```

`fnpack build` 前请确认 `cmd/*` 有执行权限、`app/ui/` 下存在图标、目录里没有 `__pycache__`。

## 七、已知限制

- 未签名包在部分 fnOS 版本可能被应用中心拒绝，需用 `appcenter-cli` 安装，或申请官方开发者证书后重新签名。
- 兼容模式是「尽力而为」的 HTML 重写：依赖 JS 动态拼绝对地址、WebSocket、或做了来源校验（CSRF/Origin 白名单）的站点，可能出现个别功能异常，此时用「浏览器标签页」打开。
- 内网自签 HTTPS 设备：兼容模式使用不校验证书的 TLS 连接，可以正常打开，但浏览器地址栏不会显示安全锁（因为走的是 NAS 的 8899 端口）。

> （注：部分内容由豆包 AI 生成）
