# Tinkercad Serial Bridge

![Tinkercad Serial Bridge 横版宣传海报](poster/tinker-serial-bridge-poster-720p.jpg)

> 为 Tinkercad Arduino 仿真建立「网页编辑器 ↔ 本地程序」的双向串口桥接。Manifest V3 浏览器扩展，**纯本地通信，不收集、不上传任何用户数据。**

[![Edge 扩展商店](https://img.shields.io/badge/Edge%20扩展商店-立即安装-5A2BE0)](https://microsoftedge.microsoft.com/addons/detail/tinkercad-serial-bridge/madkdbjchopgbjhjnpoandbmmjfajcmb)

Tinkercad 的 Arduino 仿真跑在云端网页里，而你的控制程序跑在本地。本扩展把网页里串口监视器的输出，实时转发给你本机的 Node.js 服务（或任意 HTTP 服务）；同时把服务下发的指令，写回仿真串口的「输入框 + 发送」里，从而打通云端仿真与本地程序。

---

## ✨ 功能特性

- **双向串口通信**：串口输出实时上行，外部指令实时下行写回仿真
- **增量上报**：只上报新增行，串口快速打印多行时不再只留最后一行
- **服务层统一出口**：网络请求集中在 `background.js`，避开页面 CORS 限制，便于重试与诊断
- **运行状态面板**：服务在线/离线、页面连接数、最近上下行内容与时间、页面元素检测结果
- **多标签页防抢指令**：多个 Tinkercad 标签页同时打开时，只有一个 leader 拉取指令，避免队列被互抢
- **选择器兜底**：内置多套选择器 + iframe 扫描 + 自定义选择器配置，Tinkercad 改版后无需更新插件
- **一键自检**：测试连接、手动下发指令，快速定位是服务端、网络还是页面元素的问题
- **配置持久化**：所有设置自动保存，重启浏览器不丢失

---

## 📥 安装

### 方式一 · Edge 扩展商店（推荐，普通用户）

前往商店页一键安装，无需开启开发者模式：

👉 [![Edge 扩展商店](https://img.shields.io/badge/Edge%20扩展商店-立即安装-5A2BE0)](https://microsoftedge.microsoft.com/addons/detail/tinkercad-serial-bridge/madkdbjchopgbjhjnpoandbmmjfajcmb)

也可直接在 Microsoft Edge 扩展商店搜索「Tinkercad Serial Bridge」。安装后点击工具栏图标即可打开配置面板。

### 方式二 · 加载已解压扩展（开发者 / 想改代码）

1. 将项目克隆或下载到本地
2. 打开 `edge://extensions`（Edge）或 `chrome://extensions`（Chrome）
3. 打开右上角「开发者模式」
4. 点击「加载已解压缩的扩展程序」，选择本项目文件夹
5. 改完代码后，点卡片上的「重新加载」即可生效

---

## 🚀 快速开始

1. **启动中转服务**：进入 `sample/`，运行 `node led.js`（默认监听 `http://localhost:8080`，可用 `PORT=9000 node led.js` 改端口）。
2. **打开扩展面板**：点工具栏图标，确认「服务地址」为 `http://localhost:8080`、打开「启用桥接」，点「保存设置」。
3. **运行仿真**：打开 Tinkercad Arduino 仿真项目并点运行。面板「Tinkercad 页面」显示已连接、三项「元素检测」全为 ✓ 即成功。

之后串口打印的内容会出现在你的本地服务日志里，服务下发的指令也会实时写回仿真。

---

## ⚙️ 配置项说明

点击工具栏图标打开配置面板：

- **服务地址**：默认 `http://localhost:8080`，会自动去掉结尾多余的 `/`；填写非 localhost 地址时会弹一次授权请求
- **上行间隔**：读取串口监视器的间隔，默认 800ms（串口内容变化时也会立即上报，不等轮询）
- **指令轮询间隔**：向服务端拉取待发指令的间隔，默认 2000ms
- **行过滤**：填写后只有包含该字符串的行才会上报，用于过滤仿真自带的调试输出，留空则全部上报
- **启用桥接**：总开关
- **高级 → 自定义选择器**：Tinkercad 改版导致自动探测失效时，手动填写监视器 / 输入框 / 发送按钮的 CSS 选择器

点「保存设置」立即生效，无需刷新页面。

---

## 🔌 服务端接口

插件兼容以下接口，`sample/led.js` 已全部实现，也可以用自己的服务替换。

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/send?out=<line>` | 上行一条串口输出（**原样中转**，一行一个请求） |
| GET | `/cmd` | 取出一条待发指令，队列为空返回空字符串 |
| POST | `/cmd` | 下发指令入队，支持纯文本或 `{"cmd":"..."}` |
| GET | `/getLog` | 返回最新一条串口输出（兼容旧前端） |
| GET | `/log?n=50&since=0` | 返回 JSON 环形日志 `{seq, latest, lines[]}` |
| GET | `/health` | 健康检查，供插件自检 |
| POST | `/reset` | 清空指令队列与日志 |

---

## 🔒 隐私

- 仅申请 `storage` 权限，用于保存服务地址、桥接开关、轮询间隔等**本地**配置
- 不读取浏览记录、不收集身份信息、不上传任何用户数据；所有配置仅存于本机 `chrome.storage.local`
- 仅当你把服务地址填成非 localhost 时，浏览器会弹一次授权，确认后才会向该地址发请求，且可随时撤销
