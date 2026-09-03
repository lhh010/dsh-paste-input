# dsh-paste-input

**简体中文** | [English](./README.en.md)

DSH WebUI 文件输入增强插件：**Ctrl+V 粘贴** + **全页面拖拽** + **选择文件/文件夹**，发送时复制进会话工作区临时附件目录，并把对话气泡里的附件文本块**折叠为文件 chip**。

派生自 [dsh-external/dsh-multimedia-webui-input](https://github.com/dsh-external/dsh-multimedia-webui-input)（MIT），在其基础上新增剪贴板粘贴输入、首次告知弹窗与气泡附件折叠。

> **你的 DSH 版本决定装哪个插件版本**（装错会崩：常见症状 `useConversation is not a function`）
> - DSH **0.1.1-rc.2**（npm 最新）：装**旧版** `'@dsh-external/dsh-paste-input@github:lhh010/dsh-paste-input#v0.1.5'`
> - DSH **0.1.2-alpha.1 / alpha.2 / alpha.3 / alpha.4 / alpha.5 / rc.1**：装**新版**（下方默认命令）
## 安装（profile 模式）

```sh
# 方式一：git 依赖固定 tag（公开镜像，推荐；也可用 github:lhh010/dsh-paste-input）
dsh plugin --profile web add '@dsh-community/dsh-paste-input@github:lhh010/dsh-paste-input#v0.1.18'

# 方式二：本地 link
# dsh plugin --profile web add link:/path/to/dsh-paste-input
```

并在 `~/.dsh/profiles/web/cordis.patch.yml` 追加（热重载，无需重启）：

```yaml
- insert:
    - id: dsh-paste-input
      name: '@dsh-community/dsh-paste-input'
```

> **安装提示**：pnpm 11 首次安装可能拦截 node-pty 等构建脚本——在 `~/.dsh/profiles/web` 下执行 `pnpm approve-builds --all` 放行后重跑安装命令；装完**硬刷新浏览器**（Ctrl/Cmd+Shift+R）。

### 提示词安装（让 DSH 自己装）

把下面这段提示词发给任意一个 DSH 会话，模型会替你完成安装：

> 帮我安装 dsh-paste-input 插件（DSH 文件输入增强：粘贴/拖拽文件），步骤：
> 1. 执行 `dsh plugin --profile web add '@dsh-community/dsh-paste-input@github:lhh010/dsh-paste-input#v0.1.18'`（首次可能被 pnpm 11 拦截 node-pty 构建脚本而失败）
> 2. 在 `~/.dsh/profiles/web` 下执行 `pnpm approve-builds --all`（放行构建脚本）
> 3. 再执行一次第 1 步的安装命令
> 4. 完成后在 `~/.dsh/profiles/web/cordis.patch.yml` 追加 - insert 插件行（id: dsh-paste-input，name: '@dsh-community/dsh-paste-input'），并提醒我硬刷新浏览器（Ctrl/Cmd+Shift+R）
> 遇到报错先查 https://github.com/lhh010/dsh-paste-input README 的常见问题/已知限制。

hip 全链路可用；0811 与 0812 最终快照实机 boot 验证通过（见下）。

## 迁移指南（DSH 0.1.1-rc.1 → 0.1.2-alpha.1）

> 兼容 DSH `dsh-v0.1.2-alpha.3`（无构建面，实机验证；纯客户端）

本插件**无需迁移**：运行时仅依赖 `react` 与 `slots` / `conversation` / `sessions` / `inputTriggers` 服务及 `conversation.input.left` / `conversation.input.dock` / `settings.section` 槽位——这些服务与槽位在 0.1.2-alpha.1 全部保留，客户端 API 重构（`@deepseek-ai/dsh-client-runtime` 移除、`ConversationSnapshot` 视图化、`ctx.slots.inject` 注册范式）不触及本插件的自包含 bundle；直接安装最新 tag（`#v0.1.9`）即可。

**npm 发版兼容**：兼容 DSH npm 发版 `@deepseek-ai/dsh@0.1.1-rc.1`（v0.1.5 实机 boot 验证：`dsh --profile web` 启动后 boot 清单包含本插件、`/plugins/@dsh-community/dsh-paste-input/client.js` 返回 200，依赖的 `inputTriggers`/`conversation.input` 门面与四个槽位在 0.1.1-rc.1 上保持不变）与 `@deepseek-ai/dsh@0.1.0-rc.8`（v0.1.4 实机验证，适配要点见下节），同时兼容 `@deepseek-ai/dsh@0.0.1-rc.5`（dist-tag `next`，即最终快照 snapshot0812 的 npm 发版；`npm exec -p @deepseek-ai/dsh@0.0.1-rc.5 -- dsh --profile web --port <port>` 可访问指定版本并启动，lib 生产模式）与 `@deepseek-ai/dsh@0.0.1-rc.2`（snapshot0811 的 npm 发版）。实测（npm rc.5 基线）：`dsh web` 启动后 `window.__DSH_BOOT__` 清单包含 `@dsh-community/dsh-paste-input`（inject: `dsh-client-runtime`/`dsh-client-ui-input-trigger`/`dsh-client-ui-conversation`/`dsh-client-ui-settings`），`/plugins/@dsh-community/dsh-paste-input/client.js` 返回 200；client 半经 `window.__ModuleLoader__.load` 正确注册，host 半的 `webServer` 上传路由在 rc.5 consumer 中加载成功。本插件**无任何 cordis 依赖**（无 peerDependencies；lib 构建产物无 cordis 导入）——0811 的 cordis 更名（`cordis` → `@deepseek-ai/cordis`）与本插件零影响，`npm install` 无需额外参数。

### 0.1.0-rc.8 兼容要点（npm 发版 `@deepseek-ai/dsh@0.1.0-rc.8`，v0.1.4）

- **输入机引用区间变化（修复删除失效）**：rc.8 的输入机把引用 occurrence 的行内区间从单个占位符字符改为完整显示文本（`@` + label，见 `referenceDraftText`）。v0.1.3 的 dock 删除逻辑只切除 1 个字符，点击 × 后 dock chip 消失但输入框内残留 `📎 image.png …` 之类的纯文本。v0.1.4 改用 `input.consumeToken({ kind: 'span', span: { start: occurrence.offset, end: occurrence.offset + occurrence.length, draftRev } })`（rc.8 `SessionInputShell` 的官方删除动词，带 CAS 保护）整段移除；无 `consumeToken` 的旧版本回退为按 `occurrence.length` 切除的 `setDraft`。
- **官方文件外观（修复蓝色 @ + 回形针 + 蓝色文件名）**：rc.8 的 `insertReference` 支持官方 `appearance` 字段（`file`/`folder`/`session`，官方 `@file` 引用源即用 `appearance: 'file'`），composer 内联 chip 由官方样式渲染（隐藏 @ 字形 + 官方文件图标 + 蓝色文件名）。v0.1.3 自造的 `📎 ` emoji 前缀 + 8 字符截断 label 会渲染成「蓝色 @ + 回形针 + 蓝色文件名」，v0.1.4 移除 emoji 前缀，label 改为纯文件名（与官方 @file 引用一致），并附带 `appearance: 'file'`。
- **槽位与服务不变**：`conversation.input.left` / `conversation.input.dock` / `conversation.input.right`（`kind: 'list'`, `scope: 'session'`）与 `settings.section` 在 rc.8 保持声明；`inputTriggers.registerSource`、`conversation.input.for(actx)`（返回带 `state`/`insertReference`/`consumeToken`/`setDraft` 的 facade）、host 侧 `webServer.register` 均不变。气泡折叠（`DSH_PASTE_INPUT_V1` 标记协议）与上传路由不受影响。

### 0809 兼容要点（实机验证）

- **加载机制变化**：0809 重构了客户端插件机制——旧的 `dsh.plugin.json` 清单 + `resolveClientPath`（`packages/plugin/plugin`）已删除，改为 **package.json 的 `dshClient` 声明**（`platform: 'web'`，可选 `inject`/`immediately`）+ `exports["./client"]` 指向构建产物；宿主扫描 loader 条目组成 boot 图，Web 端从 `/plugins/<id>/client.js` 拉取。本插件 package.json 已满足该声明，无需改动。
- 附件消息协议（`==== DSH_PASTE_INPUT_V1 ====` 标记）与 `.dsh/tmp/attachments/<session>/<send>/` 目录逻辑不依赖快照内部实现，0809 实测全链路成功。
- **构建要求**：0809 宿主在激活时校验 `dshClient` 包的构建产物，缺失会抛 `ClientPackageCompositionError` 并**拒绝启动 `dsh web`**——升级快照或改源码后必须重新 `pnpm run build` 再启动，否则浏览器拉到的是旧 `lib/client.js`。

### 0810 兼容要点（snapshot0810）

- **元数据发现变化**：0810 的 ClientModuleHostService 在启动时扫描已加载插件的 package.json，但只读**嵌套 `dsh.client`**（`packages/client/modules/src/index.ts` 的 `resolveMeta`，`pkg.dsh.client`）；顶层 `dshClient` 字段读不到会静默丢出 boot 图——无日志、无报错，"启动顺利但插件全没"。本插件已从顶层 `dshClient` 迁移为嵌套 `dsh.client`（inject 原样保留）；`lib/client.js` 构建产物不变（package.json 不参与编译），symlink 安装改源仓库即生效，无需重装。

### 0811 兼容要点（snapshot0811，实机验证）

- **cordis 更名对本插件零影响**：0811 将 vendored cordis 由 `cordis@4.0.0-rc.7` 更名为 `@deepseek-ai/cordis@4.0.1-rc.1`（官方 client 包随之全部改从 `@deepseek-ai/cordis` 导入）。本插件不导入 cordis（无 peerDependencies、lib 构建产物无 cordis 引用），无需任何迁移。
- **实机 boot 验证**：snapshot0811（`snapshots/20260811T152241Z`）web 启动后 `window.__DSH_BOOT__` 清单包含 `@dsh-community/dsh-paste-input`（inject: `dsh-client-runtime`/`dsh-client-ui-slash`/`dsh-client-ui-conversation`/`dsh-client-ui-settings`），`/plugins/@dsh-community/dsh-paste-input/client.js` 返回 200。本插件使用的槽位 `conversation.input.left`/`conversation.input.dock`（`ui-conversation` 声明）与 `settings.section`（`ui-settings` 声明）在 0811 上保持声明；`slash` 服务与 `window.__ModuleLoader__` 加载协议不变。

### 0812/最终快照 兼容要点（snapshots/20260812T172954Z-final，实机验证）

- **client 服务更名：`slash` → `inputTriggers`**：最终快照将输入触发服务由 `slash` 更名为 `inputTriggers`（随官方包 `@deepseek-ai/dsh-client-ui-slash` → `@deepseek-ai/dsh-client-ui-input-trigger` 一并改名，服务与 `registerSource` API 本身不变）。本插件 `lib/client.js` 已同步迁移 4 处（两个 inject 数组 + `ctx.get` + `registerSource` 调用），`dsh.client` 元数据的 inject 列表同步由 `dsh-client-ui-slash` 迁移为 `dsh-client-ui-input-trigger`。
- **host 服务更名：`httpServer` → `webServer`**：最终快照将 host 侧 HTTP 路由注册服务由 `httpServer` 更名为 `webServer`（`packages/host/webserver` 提供，`register({ kind: 'prefix', path, handler })` API 不变）。本插件 `lib/index.js` 已同步迁移 2 处（inject 数组 + `ctx.webServer.register` 调用），上传路由照常注册。
- **cordis 更名与本插件零影响**：与 0811 相同，本插件不导入 cordis（无 peerDependencies、lib 构建产物无 cordis 引用），`cordis` → `@deepseek-ai/cordis` 更名（npm rc.5 基线上为 `4.0.1-rc.4`）零影响，`npm install` 无需额外参数。
- **实机 boot 验证**：最终快照（`snapshots/20260812T172954Z-final`）web 启动后 `window.__DSH_BOOT__` 清单包含 `@dsh-community/dsh-paste-input`；npm rc.5 consumer `dsh web` 启动后 boot 清单同样包含本插件（inject 已显示 `dsh-client-ui-input-trigger`），`/plugins/@dsh-community/dsh-paste-input/client.js` 返回 200，host 半 `webServer` 上传路由加载成功。本插件使用的槽位 `conversation.input.left`/`conversation.input.dock`（`ui-conversation` 声明）与 `settings.section`（`ui-settings` 声明）在最终快照与 rc.5 上保持声明；`inputTriggers` 服务与 `window.__ModuleLoader__` 加载协议不变。

## 更新记录 / Changelog

### 2026-09-03 · v0.1.18 — 声明支持 dsh-v0.1.2-rc.1

- **验证**：alpha.5→rc.1 为纯版本号提交（252 文件零代码差异）；实机 rc.1 验证通过（悬停预览/查看器正常），无需代码改动

### 2026-09-03 · v0.1.17 — 图片/动图悬停预览 + 点击查看器（缩放/平移）

- **新功能（悬停缩略图）**：图片类附件（png/jpg/jpeg/gif/webp/bmp/avif/ico）的 chip 悬停即弹出小预览卡，GIF 动图原样播放；输入框待发送 chip（本地字节，blob URL）与气泡内已发送 chip（宿主端按所有权标记校验后回读文件）都支持
- **新功能（点击查看器）**：点击图片 chip 打开全屏查看器——滚轮以光标为中心缩放（20%–800%）、左键拖动上下左右平移、双击在 1×/2× 间切换、`+`/`-`/`0`/`Esc` 快捷键、工具栏含缩放百分比/重置/复制完整路径/关闭；GIF 在查看器中持续播放
- **宿主端新增只读路由** `GET /dsh-paste-input/v1/file?root=<发送目录>&path=<相对路径>`：仅服务**所有权标记（`.dsh-paste-input.json`）声明过的图片文件**（SVG 除外，避免同域脚本执行），路径解析约束在发送目录内，单文件 ≤64 MiB
- 非图片 chip 行为不变（悬停显示原始附件块、点击复制路径）；图片 chip 的「复制路径」移入查看器工具栏
- **修复（dock chip 崩溃，v0.1.16 遗留）**：输入框上方附件 dock 的删除按钮引用了不在本作用域的 `busy` 变量，chip 一渲染即 `ReferenceError`，整个 dock 槽位被错误边界吞掉（表现：dock 上的附件气泡消失）；已移除该悬空引用

### 2026-09-02 · v0.1.16 — 修复 AttachButton 崩溃 + 版本检查 403 改走 jsdelivr

- **修复（AttachButton 崩溃）**：`conversation.input.left` 槽位不提供 owner props（无 `input`），`props.input.phase` 读取 undefined 崩溃（Console 报 `Cannot read properties of undefined (reading 'phase')`，槽位条目被框架错误边界捕获）。改用可选链 + `'plain'` 默认值（`add()` 自身有 phase 守卫不会误操作）。dock 槽位的 `occurrences` 同样加防御
- **修复（403 刷屏）**：版本检查的 tag 源从 `api.github.com`（未认证限流 ~60 req/hr → 403）改为 `data.jsdelivr.com/v1/packages/gh/`（CDN，无限流，CORS 友好）
### 2026-09-02 · v0.1.15 — 声明支持 dsh-v0.1.2-alpha.5

- **验证**：alpha.5 为纯 bug 修复（升级路径问题），client 运行 API 无变更；lib 产物校验通过

### 2026-09-02 · v0.1.14 — 声明支持 dsh-v0.1.2-alpha.4

- **验证**：alpha.4 下 client 运行 API 无破坏性变更（changelog 仅宿主侧 Session events 重构）；lib 产物校验通过

### 2026-09-02 · v0.1.13 — 更新提示词补版本路由与排查指引

- **修复（更新提示词）**：提示词新增第 0 步（先 `dsh --version` 确认本地 DSH 版本，对照 README「版本兼容」表选对应 tag，不匹配则改装）与第 3 步（安装失败/版本不匹配/启动报错先查 README「版本兼容」「已知限制」章节）；原两步安装流程不变
### 2026-09-01 · v0.1.12 — 版本检查增加缓存 / 403 降级

- **修复（反复 403）**：GitHub tags API 在限流/未授权时返回 403，旧代码每次页面加载与点击重试都重新请求一次，console 被 403 刷屏。现按结果缓存到 localStorage：成功结果缓存 10 分钟、瞬时网络失败 60s、硬 403（限流）缓存 5 分钟——窗口内直接返回缓存结论**不再发请求**，手动「重试」仍可强制执行一次
- **降级文案**：区分「网络不可达」与「GitHub 拒绝访问（限流/403）」——后者显示「版本检查暂不可用（GitHub 拒绝访问），已缓存」，不再误导为网络问题
### 2026-09-01 · v0.1.11 — 版本检查 chip 不再被 GitHub CDN 缓存滞后误导

- **修复**：刚 push 新 tag 后的几分钟内，GitHub tags API / raw CDN 仍返回旧 tag，「已是最新版本」chip 会把**旧的远端 tag** 当作最新显示（如运行 0.1.10 却显示「已是最新 0.1.9」）。现取「拉到的 tag 与运行版本」中较新者展示；离线 chip 的重试路径同样处理
### 2026-09-01 · v0.1.10 — 修复 dock 删除失效 + 同名文件自动加序号

- **修复（删除失效 → unavailable）**：DSH 0.1.2-alpha 的输入机里 occurrence 的 offset/length 是 **clipboard 投影坐标**（chip 展开为完整 `[attachment: …]` 文本），而 `consumeToken` 的 span 校验在 **detect 投影坐标**（chip 仅占 1 个 U+FFFC 字符）下工作——旧代码直接把 clipboard 坐标传入导致替换必然失败，record 却已删除：dock chip 显示 unavailable、输入框 chip 残留。现按「前面每个 chip 缩短 length−1」精确换算成 detect 坐标再调用 `consumeToken`，失败时回退 setDraft 整段切除
- **修复（第二次粘贴报错 / 顶掉）**：`insertReference` 的插入点原来取 `snapshot.draft.length`（clipboard 投影长度），第一个附件存在后插入点越界 → `The DSH composer changed before the attachment could be inserted`。现同样按存活 chip 折算成 detect 坐标，多个附件可连续粘贴共存
- **新增（粘贴文件统一重命名）**：粘贴的文件统一改基础名——图片 `paste_image.<ext>`、其他文件 `paste_file.<ext>`（扩展名优先取原文件名，缺省按 MIME 补全）；重名自动追加 `(2)`、`(3)`… 序号（以 composer 实时 chips + records 为冲突集），改名同步进上传路径。**仅粘贴路径改名**，拖拽与文件/文件夹选择保留原始文件名
- **验证**：实机验证通过——连续粘贴两张截图得到 `paste_image.png` 与 `paste_image(2).png` 共存；dock × 删除上下同步；node --check 通过
### 2026-08-20 · v0.1.5 — 声明 DSH 0.1.1-rc.1 兼容性（实机 boot 验证）

- **验证**：DSH npm `0.1.1-rc.1` 实机 boot 验证通过——boot 清单包含本插件、client.js 返回 200；0.1.4 的 rc.8 适配（`consumeToken` 整段删除、`appearance: 'file'` 官方外观、内联 chip 整体编辑保护）在 0.1.1-rc.1 上行为无回归（所依赖的 `inputTriggers.registerSource`、`conversation.input.for` 门面与 `conversation.input.left/dock`、`settings.section` 槽位均保持不变）

### 2026-08-20 · v0.1.4 — DSH 0.1.0-rc.8 适配（删除失效 + 内联 chip 外观）

- **修复（删除失效）**：rc.8 输入机的引用 occurrence 占据 `@` + label 的完整行内区间（不再是 1 个占位符字符），v0.1.3 点击 dock chip 的 × 只删掉 `@` 一个字符，输入框内残留附件文本。现改用 rc.8 官方删除动词 `input.consumeToken`（span CAS 整段切除），旧宿主回退为按 `occurrence.length` 切除
- **新增（整体编辑保护）**：内联附件 chip 的文件名不可单独删改（部分编辑会被拦截并自动选中整个 chip，下一次按键整体删除/替换），见上「内联 chip 整体编辑保护」
- **修复（内联 chip 外观）**：移除自造的 `📎 ` emoji 前缀与 8 字符截断（在 rc.8 上渲染为「蓝色 @ + 回形针 + 蓝色文件名」），label 改为纯文件名并附带官方 `appearance: 'file'`，与官方 `@file` 引用的内联 chip 外观一致；完整路径与大小仍显示在输入框上方的 dock chip 中
- **新增（内联 chip 整体编辑保护）**：输入框内的附件 chip 只能整体删除/替换，不能改写其中的文件名——在 composer textarea 上以捕获阶段拦截 `beforeinput`，任何只触碰 chip 一部分的编辑（在 label 中间打字、退格删一个字符、选半截替换）都被阻止并把选区扩展到整个 chip（下一次按键即整体删除/替换）；已完整选中整个 chip 的删除/替换放行。守卫通过 `textarea.value === draft` 识别 composer，不影响队列编辑等其他输入框；dock 的 × 按钮仍走 `consumeToken` 整段删除
- **验证**：DSH npm `0.1.0-rc.8` 实机验证——粘贴/拖拽 → dock chip → × 删除后输入框同步清空；发送 → 复制进附件目录 → 气泡折叠 chip 全链路可用

### 2026-08-13 · v0.1.3 — 最终快照服务改名迁移（snapshot0812 + npm rc.5）

- **迁移（client）**：`slash` → `inputTriggers`（lib/client.js 4 处：inject 数组 ×2 + `ctx.get` + `registerSource` 调用），`dsh.client` 元数据 inject 由 `@deepseek-ai/dsh-client-ui-slash` 迁移为 `@deepseek-ai/dsh-client-ui-input-trigger`——最终快照将输入触发服务与官方包一并更名，服务与 `registerSource` API 不变
- **迁移（host）**：`httpServer` → `webServer`（lib/index.js 2 处：inject 数组 + `ctx.webServer.register` 调用）——最终快照将 host 侧 HTTP 路由服务更名，`register({ kind: 'prefix', path, handler })` API 不变
- **验证**：DSH 最终快照（`snapshots/20260812T172954Z-final`）与 npm rc.5（`@deepseek-ai/dsh@0.0.1-rc.5`）consumer 实机 boot 验证通过（boot 清单包含本插件、client.js 返回 200、webServer 上传路由加载成功）

### 2026-08-11 · v0.1.2 — 客户端插件元数据迁移（snapshot0810）

- **迁移**：package.json 从顶层 `dshClient` 声明迁移为嵌套 `dsh.client`（inject 原样保留）——0810 的 ClientModuleHostService 只读 `pkg.dsh.client`，旧字段会被静默忽略导致插件不进 boot 图
- **验证**：DSH snapshot0810 实机验证通过（粘贴 → 复制进附件目录 → 气泡折叠 chip 全链路）

### 2026-08-10 · v0.1.1 — 修复气泡折叠 chip 显示位置错乱

- **修复**：发送时若 chip 前后都有输入文字（尤其是多文件发送），折叠后的文件 chip 位置错乱——此前所有用户文字被合并为一个文本块堆在顶部，第一个 chip 因 flex 布局悬浮在首行文字右侧，其余 chip 散落在文本块下方；现按源顺序穿插渲染（文字 → chip → 文字 → chip…），文字段独占一行，相邻附件块的 chip 自动并排
- **修复**：折叠区文字与 chip 对齐气泡内部 16px 文字缩进（去掉此前多余的横向内缩与底部空隙）
- **验证**：DSH snapshot0809 实机验证通过

## 能力

- **Ctrl+V 粘贴**：粘贴截图/复制的图片/文件 → 作为附件加入输入框（首次粘贴弹出告知弹窗，可勾选"不再提示"，选择持久化在浏览器 localStorage）
- **全页面拖拽**：文件/文件夹拖到页面任意位置（聊天区、空白处、输入框）即加入附件；文本/链接拖拽保持浏览器默认行为
- **选择**：输入框左侧回形针按钮 → 选择文件 / 选择文件夹
- **气泡折叠**：发送后，消息气泡里冗长的附件路径文本块（含 `==== DSH_PASTE_INPUT_V1 ====` 标记协议）自动折叠为 📎 文件 chip；你在 chip 前后输入的文字按原顺序穿插保留（多文件发送时文字与各文件的 chip 逐段交错，chip 独占一行），悬停 chip 显示完整原始附件块（路径/清单/文件列表），点击 chip 复制完整路径
- **图片预览**：图片/动图附件（含 GIF）悬停 chip 弹出小预览图（动图原样播放），点击 chip 打开全屏查看器——滚轮缩放（以光标为中心，20%–800%）、拖动平移、双击 1×/2×、`+`/`-`/`0`/`Esc` 快捷键、工具栏可复制完整路径；输入框待发送 chip 与气泡内已发送 chip 均支持（宿主端按所有权标记校验后回读已发送文件）
- 发送时文件复制到 `<会话工作区>/.dsh/tmp/attachments/<session>/<send>/`，绝对路径随消息前置给模型，无权限问题
- 设置面板：附件用量统计与按会话/工作区清理（所有权标记保护，二次确认）

## 与 dsh-vision 协作：截图识别

配合 [dsh-external/dsh-vision](https://github.com/dsh-external/dsh-vision) 插件（注册 `view_image` 工具，桥接任意 OpenAI 兼容 VLM，默认智谱免费 `glm-4.6v-flash`），本插件的粘贴/拖拽截图可以**直接识别**：

1. 截图（Win+Shift+S）→ 粘贴或拖入 DSH
2. 发送后截图复制进工作区附件目录
3. 模型看到附件路径 → 调用 `view_image` → VLM 返回图片内容（OCR 提取文字、读图表、识别 UI 布局等）

两个插件零耦合：本插件负责"文件进对话"，dsh-vision 负责"看图"，通过工作区附件路径衔接。

## 附件消息协议

附件块以显式标记界定（模型可见文本，气泡折叠识别用）：

```
==== DSH_PASTE_INPUT_V1 ====
<附件根目录绝对路径>

Files: N
Manifest: .dsh-paste-input.json
Attached files (paths are relative to the root above):
- "file.txt" (2.0 KiB)
==== END DSH_PASTE_INPUT ====
```

仅支持标记格式（历史无标记消息不折叠）。标记前后各带空行，保证用户输入的文字与标记不在同一行。

## 限制

- 粘贴的文件支持因浏览器而异：**Chrome/Edge** 在 paste 事件中只提供图片（截图、复制的图片）等媒体与文本/HTML，从文件管理器"复制文件"后粘贴不会出现文件项；**Firefox** 支持粘贴文件，但同样**不提供绝对路径**。浏览器出于安全不会把本地文件路径暴露给网页，因此粘贴均以 `文件名` 作为相对路径存储——需要原路径的场景请使用**拖拽**或**选择文件/文件夹**按钮
- 单文件 ≤ 1 GiB、单次 ≤ 2 GiB、≤ 10000 文件、≤ 64 层

## 安装（profile 模式）

```sh
# 方式一：git 依赖固定 tag（公开镜像，推荐；也可用 github:lhh010/dsh-paste-input）
dsh plugin --profile web add '@dsh-community/dsh-paste-input@github:lhh010/dsh-paste-input#v0.1.18'

# 方式二：本地 link
# dsh plugin --profile web add link:/path/to/dsh-paste-input
```

并在 `~/.dsh/profiles/web/cordis.patch.yml` 追加（热重载，无需重启）：

```yaml
- insert:
    - id: dsh-paste-input
      name: '@dsh-community/dsh-paste-input'
```

> **安装提示**：pnpm 11 首次安装可能拦截 node-pty 等构建脚本——在 `~/.dsh/profiles/web` 下执行 `pnpm approve-builds --all` 放行后重跑安装命令；装完**硬刷新浏览器**（Ctrl/Cmd+Shift+R）。

### 提示词安装（让 DSH 自己装）

把下面这段提示词发给任意一个 DSH 会话，模型会替你完成安装：

> 帮我安装 dsh-paste-input 插件（DSH 文件输入增强：粘贴/拖拽文件），步骤：
> 1. 执行 `dsh plugin --profile web add '@dsh-community/dsh-paste-input@github:lhh010/dsh-paste-input#v0.1.18'`（首次可能被 pnpm 11 拦截 node-pty 构建脚本而失败）
> 2. 在 `~/.dsh/profiles/web` 下执行 `pnpm approve-builds --all`（放行构建脚本）
> 3. 再执行一次第 1 步的安装命令
> 4. 完成后在 `~/.dsh/profiles/web/cordis.patch.yml` 追加 - insert 插件行（id: dsh-paste-input，name: '@dsh-community/dsh-paste-input'），并提醒我硬刷新浏览器（Ctrl/Cmd+Shift+R）
> 遇到报错先查 https://github.com/lhh010/dsh-paste-input README 的常见问题/已知限制。
## License

MIT（含 dsh-multimedia-webui-input 派生声明）

