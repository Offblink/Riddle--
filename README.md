# Riddle--

[MaximeRivest/riddle](https://github.com/MaximeRivest/riddle) 的浏览器重生版。不需要 e-ink 硬件，一个浏览器、一支笔（或鼠标）、一个 API Key 就够了。

## 与 riddle 的差异

| | **riddle（原版）** | **riddle--** |
|---|---|---|
| **平台** | reMarkable Paper Pro（e-ink, aarch64） | 任何浏览器（Windows / macOS / Linux / 手机） |
| **语言** | Rust + C/C++ | 单 HTML 文件 + 原生 JS |
| **显示后端** | 直接驱动 e-ink（quill）或 X11（qtfb） | Canvas |
| **笔输入** | evdev，4096 级压感 | Pointer events（鼠标 + 触摸） |
| **手写识别** | **Vision LLM**（GPT-4o 读整页 PNG 截图） | **Google IME 笔画 API**（发坐标数组，不发图片） |
| **回复生成** | 同一个 Vision 模型（读图 + 写文） | **纯文本 LLM**（DeepSeek / OpenAI 兼容） |
| **手写合成** | 字体 → Zhang-Suen 骨架化 → 笔画追踪 → 动画回放 | Canvas `fillText` + clip-reveal 动画，居中排版 |
| **个性化** | 固定字体 | 可选字体（预设 + 系统字体）、大小、速度 |
| **安装部署** | 开发者模式 + SSH + AppLoad | `python -m http.server` |
| **离线** | 支持（pi 后端） | 不支持（需 API） |
| **每次交互成本** | ~500–1000 vision tokens | ~50 text tokens（*便宜约 10–20 倍*） |

## 为什么不用 Vision LLM？

原版 riddle 把整页 PNG 截图发给 GPT-4o，让一个通用多模态大模型同时做"看图识字"和"对话生成"。500+ KB 的图片数据，其实只需要几个字节的笔画坐标。

Riddle-- 走的是**各司其职**的路线：

```
Canvas 笔画 → Google IME API（笔画→文字）→ 纯文本 LLM → Canvas 动画
```

- 识别用专门的、免费的笔画识别服务，不用 $20/Mtok 的通才
- LLM 只看文字——你付的是词不是像素
- 架构可拆卸：随时换识别后端、随时换 LLM，互不耦合

## 快速开始

```bash
# 1. 复制配置文件
cp config.example.js config.js

# 2. 编辑 config.js，填入 API Key
#    （DeepSeek、OpenAI 或任何兼容服务）
# 3. 启动本地服务器（避免 file:// 的 CORS 限制）
python -m http.server

# 4. 浏览器打开 http://localhost:8000
```

也可以在应用内点 ⚙ 修改设置，会存到 localStorage。

## 项目结构

```
riddle--/
├── index.html          # 整个应用（无构建、除 config.js 外零依赖）
├── config.js           # API Key（gitignore，不提交）
├── config.example.js   # 配置模板（可安全提交）
├── Monocraft.ttf       # UI 字体（Minecraft 像素风等宽）
├── .gitignore
├── README.md           # 中文说明（本文件）
└── README_EN.md        # English version

## 操作

| 操作 | 效果 |
|---|---|
| 写字，停笔 2.8 秒 | 用户墨迹 + 旧回复一起渐隐，日记回复 |
| Ctrl+Z 或点 ↩ | 撤销最后一笔 |
| 点 ⚙ | 打开设置面板 |
| 点右下角 ▷ | 切换调试日志 |

## 设置面板

打开 ⚙ 可调整以下项目（自动保存到本地）：

| 设置 | 说明 |
|---|---|
| 手写语言 | 中/英/日/韩等 30+ 种 |
| 回复字体 | 9 种预设（马善政、Caveat、楷体等）+ 自动加载系统字体 |
| 字体大小 | 20–48px（自动缩放适配画布） |
| 显示速度 | `−` / `+` 调节回复逐字动画速度（50–500ms），主画布实时预览 |
| API Key / Base / Model | LLM 配置（config.js 值优先） |
| 角色设定 | System Prompt |

退出设置面板时，主画布预览内容会自动渐隐。
## 手写识别后端


默认：**Google IME 手写识别 API**（内联自 [handwriting.js](https://github.com/ChenYuHo/handwriting.js)，MIT 协议）。

接收笔画坐标数组，返回识别文字。支持语言：中文简/繁、英文、日文、韩文等 30+ 种。

> ⚠️ Google 服务在国内可能需要 fān qiáng。识别模块设计为可替换——修改 `recognizeHandwriting()` 函数即可切换后端。

### 备选方案

- **百度/腾讯手写 OCR** — 把笔画渲染成小图发 OCR API。不是笔画原生，但仍比 Vision LLM 便宜 20 倍。
- **本地 ONNX 模型** — 训练/转换一个小型笔画序列分类器，用 ONNX Runtime Web 在浏览器内跑。零 API 调用。（远期规划）
## LLM 后端

默认：**DeepSeek**（`deepseek-v4-flash`）。任何 OpenAI 兼容 API 都能用——改 `config.js` 即可。

## 致谢

### [handwriting.js](https://github.com/ChenYuHo/handwriting.js) — 手写识别

由 ChenYuHo 开发（MIT 协议，185 ★）。封装了 Google 输入工具的手写识别 API——
将笔画坐标数组 `[[x1,x2,…], [y1,y2,…], []]` 直接 POST 到 Google IME 端点，
返回识别出的文字候选列表。支持中文简/繁、日文、韩文、英文等 40+ 种语言。

Riddle-- 将其核心识别逻辑（约 40 行）内联到了 `index.html` 中，去掉了对原始 Canvas
封装层的依赖，改为 `recognizeHandwriting(trace, language)` 这一简洁接口。

### [Monocraft](https://github.com/IdreesInc/Monocraft) — UI 字体

由 IdreesInc 开发（SIL Open Font License 1.1，11K ★）。一款基于 Minecraft
界面字体的等宽编程字体——1500+ 个字形，每个都在原始像素风格基础上重新调整了
间距和可读性，包含连字（ligature）支持。

Riddle-- 取用了其发布的预编译 `.ttf` 文件作为 UI 字体——状态栏、设置面板、
日志面板全部使用 Monocraft，与 Canvas 上的马善政手写体形成冷/暖对比。

## 许可证

MIT
