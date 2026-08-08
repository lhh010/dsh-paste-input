# dsh-paste-input

DSH WebUI 文件输入增强插件：**Ctrl+V 粘贴** + **全页面拖拽** + **选择文件/文件夹**，发送时复制进会话工作区临时附件目录，并把对话气泡里的附件文本块**折叠为文件 chip**。

派生自 [dsh-external/dsh-multimedia-webui-input](https://github.com/dsh-external/dsh-multimedia-webui-input)（MIT），在其基础上新增剪贴板粘贴输入、首次告知弹窗与气泡附件折叠。

## 能力

- **Ctrl+V 粘贴**：粘贴截图/复制的图片/文件 → 作为附件加入输入框（首次粘贴弹出告知弹窗，可勾选"不再提示"，选择持久化在浏览器 localStorage）
- **全页面拖拽**：文件/文件夹拖到页面任意位置（聊天区、空白处、输入框）即加入附件；文本/链接拖拽保持浏览器默认行为
- **选择**：输入框左侧回形针按钮 → 选择文件 / 选择文件夹
- **气泡折叠**：发送后，消息气泡里冗长的附件路径文本块（含 `==== DSH_PASTE_INPUT_V1 ====` 标记协议）自动折叠为 📎 文件 chip；你在 chip 前后输入的文字原样保留；悬停 chip 显示完整原始附件块（路径/清单/文件列表），点击 chip 复制完整路径
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
dsh plugin --profile web add link:E:\deepseek-harness\dsh-paste-input
# 并在 ~/.dsh/profiles/web/cordis.patch.yml 追加：
# - insert:
#     - id: dsh-paste-input
#       name: '@dsh-community/dsh-paste-input'
```

重启 `dsh web` 生效。

## License

MIT（含 dsh-multimedia-webui-input 派生声明）
