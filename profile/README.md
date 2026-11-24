<div align="center">

# CogX

**万物皆插件，数据皆本地。**
_Plugin-Everything, Local-First Desktop Dock._

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Engine](https://img.shields.io/badge/tech_stack-Rust_2024_+_Tauri_2-orange.svg)](https://tauri.app)
[![Protocol](https://img.shields.io/badge/protocol-cogx://-brightgreen.svg)](#)
[![Official Website](https://img.shields.io/badge/website-cogx.run-6366f1.svg)](https://cogx.run)

[官网 (cogx.run)](https://cogx.run) • [架构与开发文档](https://github.com/cogx-run) • [系统深链 `cogx://`](#)

</div>

---

### 💡 什么是 CogX？

**CogX 是一个以任意本地目录为工作区、万物皆插件、数据皆本地的桌面扩展坞。**

它不造私有数据黑盒，也不取代你习惯的主力工具。无论是工程仓库、笔记库还是普通文件夹，CogX 都能直接挂载，在屏幕一角提供轻量能力与陪伴。

```text
[ 你的本地目录 (代码工程 / 知识库 / 个人文件夹) ]
  ├── .cogx/           <- CogX 专属目录：保存插件代码、设置与运行数据
  ├── .git/            <- 兼容 Git：工作区文件由你管理，可自由提交与追踪
  ├── .obsidian/       <- 兼容 Obsidian：直接挂载现有知识库，共用笔记文件
  ├── .vscode/         <- 兼容 VS Code：直接挂载项目根目录，保留原有工作方式
  ├── daily-notes.md   <- 开放格式文件：插件与外部编辑器共同使用
  └── tasks.json       <- 数据透明落盘：可随时迁移、备份或批量处理
```

- 📂 **目录即工作区**：任意合法本地文件夹都能挂载。可与 Git、VS Code、Obsidian 等工具共用目录，也可用网盘或 U 盘备份、同步。
- 🧩 **一切皆插件**：时钟、番茄钟、便签、网页小窗、桌宠和游戏都由插件承载，内核不绑定具体业务。
- 🤖 **AI 生成即用**：让任意大模型按提示词生成 `main.js`，粘贴板一键导入运行。
- 🛡️ **克制清晰的微内核**：杜绝远程代码注入与动态 `eval`。能禁用的全部禁用，必须开放的能力走 Rust API，网络按域名授权；无法完全禁用的风险，通过 Rust 静态扫描披露。

---

### 🏗️ 核心架构

CogX 将系统能力与具体业务分离，控制中心、系统深链和 CLI 共用同一个 CogX core：

```text
控制中心 UI (单例)    系统深链 (cogx://)     CLI 工具 (cogx)
    │                    │                     │
    └────────────────────┼─────────────────────┘
                         ▼
               [ CogX Core ]
      ├── 权限注册与令牌校验 (Unified plugin_call)
      ├── 按域授权网络网关 (ctx.fetch)
      ├── 目录边界安全防护 (Strict Folder Confinement)
      └── Rust 计时器 (Monotonic Timer)
                         │
         ┌───────────────┴───────────────┐
         ▼                               ▼
   [ 主窗口内 ]                    [ 独立窗口 ]
 共用主窗口 WebView · 导航切换      独立窗口 · 独立 WebView
```

---

### ⚡ 插件能力标准 (ctx API)

**API 设计以降低开发者心智负担为第一准则。** 插件只需通过宿主注入的 `ctx` 获取清晰、统一的能力，符合现代Web标准。

`main.js` 示例：

```js
export default async function (ctx) {
  // 1. 本地 DOM 挂载 (使用封闭 ShadowRoot)
  const doc = ctx.root.ownerDocument;
  const btn = doc.createElement('button');
  btn.textContent = '记录一次专注';
  ctx.root.appendChild(btn);
  // 2. 写入工作区文件 (需经用户授权)
  btn.onclick = async () => {
    await ctx.workspace.write('last-focus.txt', `${new Date().toISOString()} 完成了一次专注\n`);
    await ctx.storage.set('last_focus', Date.now());
    await ctx.toast('已写盘！');
  };
  // 3. Rust 计时 (遮挡、最小化不影响计时真值)
  const timer = await ctx.timer.after(25 * 60 * 1000, async () => {
    await ctx.toast('番茄钟时间到！');
  });
  // 4. 清理任务
  ctx.onUnmount(async () => {
    await timer.cancel();
  });
}
```

---

### 📚 文档体系

- **产品白皮书**：最终架构畅想、能力边界、市场格局、竞品经验与长期影响力。
- **发展规划**：唯一维护实施阶段、验证目标、商业安排和启动条件。
- **架构与实现**：CogX core、Tauri 窗口、控制中心、CLI 与当前交付验收。
- **API 说明**：插件目录、字段、`ctx`、权限、错误码和兼容规则的唯一契约。
- **工程结构**：仓库归属、命名、测试、构建与发布规则。
- **安全隐患**：风险、最坏影响、防治和取舍的待验证清单。

---

### 📜 开源协议与品牌说明

- **微内核底座、插件标准与文档**：基于 [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0) 协议开源。可以自由学习、修改并构建自己的发行版与插件源。
- **品牌与官方资产保留**：“CogX”、机械伴鸟“考格 (Cog)”形象、`cogx.run` 域名以及官方源资产由创始人与运营主体独立保留，开源不包含商标与相关原创美术授权。

<div align="center">

**一枚齿轮咬合万千插件，一羽铜鸟陪伴每个专注日常。**

</div>
