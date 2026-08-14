# 串口 HEX 工具 · Web 版

单文件 HTML（`index.html`），用浏览器直接收发真实串口数据，支持导入 **JSON 命令表**（含串口配置）点选发送。无需安装，Chrome/Edge 打开即用，适用于嵌入式开发、硬件调试、协议验证、物联网设备测试。

> 命令表可由页面「下载组包 Skill」下载的 skill，装到 AI 助手后按协议文档自动生成。

---

## 功能一览

### 连接设置
- 枚举 / 选择已授权串口、刷新、波特率（1200 ~ 2000000，默认 115200）
- 打开 / 关闭串口，状态栏实时显示 `已连接 … @ 波特率 | RX: N B | TX: N B`
- 8N1 固定（8 数据位、无校验、1 停止位）
- 浏览器不支持 Web Serial 时顶部醒目提示，命令库功能仍可用

### 命令库（JSON）
- **导入文件 / 粘贴导入 / 导出 / 清空 / 搜索**
- 每条命令 HEX 与中文含义分行紧凑显示，HEX 自动规范化（移除非 hex、校验偶数位与字节合法性、转大写空格分隔）
- 单击选中，**双击「填入发送区」**，或点「直接发送」一键发
- 导入时若 JSON 含 `serial.baudRate`，自动切换波特率下拉框
- 导出为 JSON（含串口配置 + 元信息往返保留）
- 「下载组包 Skill」一键下载 AI 组包 skill

### 接收区
- 显示开关：**HEX / ASCII / 时间戳 / 暂停显示**（胶囊 toggle，可任意组合）
- **原始流 + 空闲合并**：串口每次 `read()` 只返回当前缓冲区字节，回环 / 高速设备会把一次数据拆成多块；工具把短时间（默认 40ms）内连续到达的字节攒到一起，等空闲再合并成一行，避免显示打成碎片
- **导出日志**：一键导出全部收发记录（TX + RX）为 `serial_log_时间戳.txt`，含连接信息与收发统计；即使中途「暂停显示」，实际收发的数据仍完整留存
- 清空接收

### 发送区
- **HEX / ASCII 切换发送**（默认 HEX）：HEX 按十六进制解析；ASCII 按 UTF-8 编码原文发送（如 `AT+RST`）
- 发送 / 清空，`Ctrl+Enter` 发送、`Ctrl+F` 搜索
- 从命令库填充时自动切回 HEX 模式（命令库均为 HEX）

### 主题
- 浅色 / 深色一键切换（右上角），记忆到 localStorage，默认深色；视觉风格对齐 shadcn neutral 主题

---

## 浏览器要求（重要）

串口收发依赖 **Web Serial API**，必须满足：

1. **Chrome** 或 **Edge**（Firefox / Safari 不支持）
2. 通过 **`http(s)://`** 或 **`localhost`** 打开 —— **不能直接双击 `index.html`**（`file://` 下 Web Serial 被禁用）
3. 首次「打开串口」时浏览器弹窗，需**手动选择并授权**对应串口

> 不满足时页面顶部显示提示，但命令库的导入 / 管理 / 搜索 / 导出仍可正常使用（只是不能收发串口）。

---

## 使用（本机）

双击 `run.bat`，或手动起一个静态服务：

```powershell
cd D:\ZZWORK\FOR_CODING\4_hex-serial-tool-web
py -3 -m http.server 8080
```

浏览器访问 `http://localhost:8080`，点击「打开串口」选择设备。

---

## 命令表文件格式（JSON）

导入 / 导出均为 **JSON**（UTF-8、标准 JSON）。结构如下：

```json
{
  "format": "serial-command-table",
  "version": 1,
  "title": "设备名 + 协议版本（可选）",
  "serial": { "baudRate": 9600, "dataBits": 8, "parity": "none", "stopBits": 1 },
  "checksum": "xor",
  "note": "校验 / 备注（可选，工具不解析）",
  "commands": [
    { "hex": "AA 55 01 00", "meaning": "查询设备状态" },
    { "hex": "55 AA 02 00 01 30 00 00 32", "meaning": "模块查询设备能力" }
  ]
}
```

| 字段 | 必填 | 说明 |
|------|------|------|
| `format` | 是 | 固定 `"serial-command-table"`，工具据此识别 |
| `version` | 是 | 固定 `1` |
| `title` | 否 | 命令表标题，建议「设备名 + 协议版本」 |
| `serial.baudRate` | 强烈建议 | 波特率整数；**导入时自动切换波特率下拉框** |
| `serial.dataBits/parity/stopBits` | 否 | 默认 `8 / none / 1`；导出原样回写，网站固定 8N1 |
| `checksum` / `note` | 否 | 人读，工具不解析；导入后导出原样回写 |
| `commands[].hex` | 是 | 字节用空格分隔，大小写不限，自动规范化为大写空格分隔 |
| `commands[].meaning` | 是 | 含义，不能为空；建议带方向前缀如 `[模组下发]`/`[MCU回复]` |

- **兼容旧格式**：也支持旧版 `HEX | 含义` 文本导入（分隔符优先级 `|` > `Tab` > `,`，`#` 注释行忽略）。
- 命令表可由「下载组包 Skill」下载的 skill 装到 AI 助手后，按协议文档（帧格式、FuncID、校验方式、波特率）自动生成。

---

## 串口参数

8N1（8 数据位、无校验、1 停止位）。波特率可选 1200 ~ 2000000。

---

## 部署 GitHub Pages

1. 新建 GitHub 仓库
2. 将本目录 **`index.html`** 推到仓库根目录（或 `docs/index.html`）
3. 仓库 **Settings → Pages** → Source: Deploy from a branch → Branch: `main` / `root`
4. 访问 `https://<用户名>.github.io/<仓库名>/`

> GitHub Pages 是 https，满足 Web Serial 的安全要求。同事用 Chrome/Edge 打开链接、授权串口即可用。

---

## 隐私说明

所有命令表文件、串口数据均在浏览器本地处理，不上传任何服务器。

## 目录结构

```
4_hex-serial-tool-web/
├─ index.html              # 单文件工具（全部 UI + 逻辑）
├─ run.bat                 # 一键起本地静态服务（端口 8080）
├─ README.md               # 本文档
└─ PROJECT/                # 示例协议 / 命令表（按设备分目录）
```
