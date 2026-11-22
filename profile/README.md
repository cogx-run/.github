<div align="center">

# CogX

**万物皆插件，数据皆本地。**
_Plugin-Everything, Local-First Desktop Dock._

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
[![Engine](https://img.shields.io/badge/tech_stack-Rust_2024_+_Tauri_2-orange.svg)](https://tauri.app)
[![Protocol](https://img.shields.io/badge/protocol-cogx://-brightgreen.svg)](#)
[![Official Website](https://img.shields.io/badge/website-cogx.run-6366f1.svg)](https://cogx.run)

[官网 (cogx.run)](https://cogx.run) • [开发规范与文档](https://github.com/cogx-run) • [系统深链 `cogx://`](#)

</div>

---

### 💡 什么是 CogX？

**CogX 是一个以自定义任意本地目录为工作区、万物皆插件、数据皆本地的桌面扩展坞。**

它不造私有数据黑盒，更不试图取代你早已习惯的主力生产力工具。你只需指定电脑中的 **任意一个文件夹** ——无论是一个正在开发的工程仓库、一个沉淀多年的笔记库，还是随手的日常目录，CogX 都能直接“附着”上去，在屏幕一角为你提供即插即用的轻量能力与陪伴。

```text
[ 你的本地目录 (代码工程 / 知识库 / 个人文件夹) ]
  ├── .cogx/           <- CogX 专属目录：收敛插件代码、运行数据、授权项与工作区独立配置
  ├── .git/            <- 完美兼容 Git：插件配置、运行数据与笔记一同提交，版本掌控在手
  ├── .obsidian/       <- 兼容 Obsidian：直接挂载为知识库，便签与双链笔记无缝协作
  ├── .vscode/         <- 兼容 VS Code：直接挂载项目根目录，构建监控与 API 盯盘随手即开
  ├── daily-notes.md   <- 你的开放格式文件：插件与外部编辑器共享，绝不私有加密
  └── tasks.json       <- 数据透明落盘：可随时迁移、备份，或交给外部脚本批量处理
```

- 📂 **目录即工作区 (Folder-as-Workspace)**：任何本地文件夹都能一键挂载为工作区，插件读写局限于根目录内，数据随目录自由迁移、备份或受版本控制系统（Git）管理。可以支持网盘，U盘进行同步与保存。
- 🧩 **一切皆插件**：时钟、番茄钟、系统状态盯盘、网页小窗、桌面旅人陪伴桌宠、各种卡牌游戏——内核零业务绑定，全部能力由插件承载。
- 🤖 **AI 代码即用即走**：让任意大模型按提示词标准生成单文件 `main.js`，不用编程剪贴板一键即导入本地即刻运行。
- 🛡️ **克制清晰的微内核**：Rust + Tauri 2 驱动。默认禁用、权限全关；杜绝远程代码注入与动态 `eval`；网络请求走严苛白名单网关。

---

### 🏗️ 核心架构

CogX 将宿主系统能力与具体业务彻底剥离，控制中心与 CLI 共享统一的 Rust core 内核：

```text
控制中心 UI (单例)    系统深链 (cogx://)     CLI 工具 (cogx)
    │                    │                     │
    └────────────────────┼─────────────────────┘
                         ▼
               [ CogX Rust Core ]
      ├── 权限注册与令牌校验 (Unified plugin_call)
      ├── 本地网络放行网关 (ctx.fetch)
      ├── 目录边界安全防护 (Strict Folder Confinement)
      └── 单调高精度计时器 (Monotonic Timer)
                         │
         ┌───────────────┴───────────────┐
         ▼                               ▼
   [ Dock 模式 ]                   [ Solo 模式 ]
 共享窗口 · 导航切换              独立 WebView · 强隔离
```

---

### ⚡ 插件能力标准 (ctx API)

每个插件就是一个极简的 Web 标准模块，通过宿主注入的受控 `ctx` 获取能力：

`main.js` 文件Demo如下：

```js
export default async function (ctx) {
  // 1. 本地 DOM 挂载 (使用封闭 ShadowRoot)
  const doc = ctx.root.ownerDocument;
  const btn = doc.createElement('button');
  btn.textContent = '记录一次专注';
  ctx.root.appendChild(btn);

  // 2. 本地工作区文件直读直写 (需经用户授权)
  btn.onclick = async () => {
    await ctx.workspace.write('focus.log', `${new Date().toISOString()} 完成了一次专注\n`);
    await ctx.storage.set('last_focus', Date.now());
    await ctx.toast('已保存到工作区目录！');
  };

  // 3. 高精度单调计时 (窗口遮挡/最小化不受浏览器策略节流)
  const timer = await ctx.timer.after(25 * 60 * 1000, async () => {
    await ctx.toast('番茄钟时间到！');
  });

  // 4. 清理注销
  ctx.onUnmount(async () => {
    await timer.cancel();
  });
}
```

---

### 🗺️ 演进路线图

- [x] **阶段一：插件底座与微内核**
      Rust core、目录工作区、权限隔离、统一 CLI、网页小窗、`cogx://` 协议与单调计时。
- [ ] **阶段二：《莫尔：桌面旅人》与游戏单品验证**
      以首款买断制叙事桌宠游戏跑通底座能力，在 Steam 平台完成独立验证。
- [ ] **阶段三：市场上线与生态闭环**
      开源 CogX 免费本体，上线官方免费源、第三方源支持与 Steam 创意工坊支持。
- [ ] **阶段四：增值服务与生态分发**
      按需上线官网独立版账号体系、云同步与精选插件源。
- [ ] **阶段五：多端工作区流转**
      移动端支持（Tauri 2 Mobile）与局域网 P2P 目录同步。

---

### 📜 开源协议与品牌说明

- **微内核底座与插件规范**：基于 [Apache License 2.0](LICENSE) 协议完全开源。任何人均可自由学习、修改并构建自己的发行版与插件源。
- **品牌与官方资产保留**：“CogX”、机械伴鸟“考格 (Cog)”形象、`cogx.run` 域名以及官方源资产由创始人与运营主体独立保留，开源不包含商标与相关原创美术授权。

<div align="center">

**一枚齿轮咬合万千插件，一羽铜鸟陪伴每个专注日常。**

</div>
