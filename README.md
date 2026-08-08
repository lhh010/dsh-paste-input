# dsh-paste-input

DSH WebUI 文件输入增强插件：**Ctrl+V 粘贴** + **拖拽** + **选择文件/文件夹**，发送时复制进会话工作区临时附件目录。

派生自 [dsh-external/dsh-multimedia-webui-input](https://github.com/dsh-external/dsh-multimedia-webui-input)（MIT），在其基础上新增剪贴板粘贴输入与首次告知弹窗。

## 能力

- **Ctrl+V 粘贴**：粘贴截图/复制的图片/文件 → 作为附件加入输入框（首次粘贴弹出告知弹窗，可勾选"不再提示"，选择持久化在浏览器 localStorage）
- **拖拽**：文件/文件夹拖入输入框区域即加入附件
- **选择**：输入框左侧回形针按钮 → 选择文件 / 选择文件夹
- 发送时文件复制到 `<会话工作区>/.dsh/tmp/attachments/<session>/<send>/`，绝对路径随消息前置给模型，无权限问题
- 设置面板：附件用量统计与按会话/工作区清理（所有权标记保护，二次确认）

## 限制

- 浏览器剪贴板只能拿到位图/文本；从文件管理器"复制文件"后粘贴无法取得路径（需用拖拽）
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
