# 🗣️ Interactive Feedback MCP

一个简单的 [MCP Server](https://modelcontextprotocol.io/)，用于在AI辅助开发工具（如 [Cursor](https://www.cursor.com)、[Cline](https://cline.bot) 和 [Windsurf](https://windsurf.com)）中实现人机协作工作流。该服务器允许您轻松地直接向AI代理提供反馈，弥合AI与您之间的差距。

**注意：** 此服务器设计为与MCP客户端（例如Cursor、VS Code）在本地一同运行，因为它需要直接访问用户的操作系统以显示UI和执行键盘/鼠标操作。

## 🖼️ 示例

![Interactive Feedback Example](https://raw.githubusercontent.com/pawaovo/interactive-feedback-mcp/main/.github/example.png)
*(请注意，示例图片可能未反映最新的UI调整，但核心交互流程保持不变)*

## 💡 为何使用此工具？

在像Cursor这样的环境中，您发送给LLM的每个提示都被视为一个独立的请求——每个请求都会计入您的每月限额（例如，500个高级请求）。当您迭代模糊指令或纠正被误解的输出时，这会变得效率低下，因为每次后续澄清都会触发一个全新的请求。

此MCP服务器引入了一种变通方法：它允许模型在最终确定响应之前暂停并请求澄清。模型不会直接完成请求，而是触发一个工具调用 (`interactive_feedback`)，打开一个交互式反馈窗口。然后，您可以提供更多细节或要求更改——模型会继续会话，所有这些都在单个请求内完成。

从本质上讲，这只是巧妙地利用工具调用来推迟请求的完成。由于工具调用不计为单独的高级交互，因此您可以在不消耗额外请求的情况下循环执行多个反馈周期。

简而言明，这有助于您的AI助手在猜测之前请求澄清，而不会浪费另一个请求。这意味着更少的错误答案、更好的性能和更少的API使用浪费。

- **💰 减少高级API调用：** 避免浪费昂贵的API调用来基于猜测生成代码。
- **✅ 更少错误：** 行动前的澄清意味着更少的错误代码和时间浪费。
- **⏱️ 更快周期：** 快速确认胜过调试错误的猜测。
- **🎮 更好协作：** 将单向指令转变为对话，让您保持控制。

## 🌟 核心功能与最新改进

### 1. 交互式反馈窗口
   - 当AI助手需要澄清或在完成任务前需要您的确认时，会弹出一个UI窗口。
   - 您可以在此窗口中输入文本反馈。
   - 如果AI助手提供了预定义选项，您可以直接勾选。

### 2. 图片处理与直接对话模式
   - **粘贴图片和文本：** 您可以直接在反馈输入框中粘贴图片（例如，使用Ctrl+V）。支持同时粘贴文本和多张图片。
   - **智能发送逻辑：**
     - 当反馈中包含图片时，系统会进入"直接对话模式"。
     - 点击"发送"后，MCP窗口将关闭，并自动激活Cursor的聊天输入框。
     - 您在UI窗口中输入的所有内容（包括文本和所有图片）将作为 **一次完整的输入** 被自动注入到Cursor聊天框中，然后自动按下Enter键发送。
     - 此过程经过优化，确保文本和图片按预期顺序、一次性完整发送，避免了之前可能出现的分次发送问题。
   - **图片预览：** 粘贴的图片会在输入框下方显示缩略图预览。您可以点击预览图将其移除。
   - **依赖项：** 此功能依赖 `pyperclip`、`pyautogui`、`Pillow` 和 `pywin32` (仅Windows)。

### 3. 常用语管理
   - 您可以保存和管理常用的反馈短语，以便快速插入。
   - 通过"常用语"按钮访问此功能。

### 4. UI和体验优化
   - **输入框优化：** 修复了长按BackSpace键删除文字时的卡顿问题，提供更流畅的输入体验。
   - **选项复制：** 现在可以方便地从预定义选项的文本标签中复制文本。
   - **界面调整：** 顶部提示文字区域高度增加到200px，以更好地显示提示信息。

## 🛠️ 工具

此服务器通过模型上下文协议 (MCP) 公开以下工具：

- `interactive_feedback`:
    - **功能：** 向用户提出问题，并可提供预定义的选项供用户选择。返回用户的文本输入和/或选中的选项。
    - **参数：**
        - `message` (str): 要向用户显示的问题或提示。
        - `predefined_options` (List[str], 可选): 一个字符串列表，每个字符串代表一个用户可以选择的预定义选项。
    - **返回：** 一个包含用户反馈的JSON对象，格式如下：
      ```json
      {
        "content": [
          {"type": "text", "text": "用户的文本反馈和/或选中的选项组合文本"},
          // 如果有图片，这里可能还会有图片相关的 ContentItem
          // 但在当前实现中，图片通过直接对话模式发送，MCP返回通常为空
        ]
      }
      ```
      **注意：** 当使用图片并通过直接对话模式发送时，此工具调用本身可能只返回一个空内容，因为实际的反馈（文本和图片）已直接注入到Cursor聊天框中。

## 📦 安装

1.  **先决条件：**
    *   Python 3.11 或更新版本。
    *   [uv](https://github.com/astral-sh/uv) (一个快速的Python包安装和解析工具)。按以下方式安装：
        *   Windows: `pip install uv`
        *   Linux/macOS: `curl -LsSf https://astral.sh/uv/install.sh | sh`
        *   或者参考 `uv` 官方文档获取其他安装方式。

2.  **获取代码：**
    *   克隆此仓库：
        `git clone https://github.com/pawaovo/interactive-feedback-mcp.git`
    *   或者下载源代码压缩包并解压。

3.  **安装依赖：**
    *   进入仓库目录 (`cd interactive-feedback-mcp`)。
    *   运行：
        `uv pip install -r requirements.txt`
    *   **图片支持的额外依赖：** 为了使图片粘贴和直接对话模式正常工作，还需要以下包：
        `pyperclip`, `pyautogui`, `Pillow`。
        在Windows上，还需要 `pywin32`。
        这些通常可以通过 `uv pip install pyperclip pyautogui Pillow pywin32` (Windows) 或 `uv pip install pyperclip pyautogui Pillow` (其他系统) 来安装。`requirements.txt` 已包含这些。

## ⚙️ 配置

1.  将以下配置添加到您的 `claude_desktop_config.json` (Claude Desktop 旧版) 或 `mcp_servers.json` (Cursor, 通常在 `.cursor-ai/mcp_servers.json` 或用户配置目录中)：

    **重要提示：** 将 `/path/to/interactive-feedback-mcp` 替换为您在系统上克隆或解压本仓库的 **实际绝对路径**。

    ```json
    {
      "mcpServers": {
        "interactive-feedback": {
          "command": "uv",
          "args": [
            "run",
            "python", // 确保明确使用python来运行
            "server.py"
          ],
          "cwd": "/path/to/interactive-feedback-mcp", // 指定工作目录
          "timeout": 600,
          "autoApprove": [ // 自动批准此工具的调用，避免每次都需手动确认
            "interactive_feedback"
          ]
        }
      }
    }
    ```
    **关于 `command` 和 `args` 的说明:**
    - 如果 `uv` 在您的系统路径中，并且您希望 `uv` 管理虚拟环境和运行脚本，可以使用 `"command": "uv", "args": ["run", "python", "server.py"]`。
    - 如果您更倾向于直接使用系统Python（并已在全局或项目虚拟环境中安装了依赖），可以使用 `"command": "python", "args": ["server.py"]` (或python3)。
    - **`cwd` (Current Working Directory):** 强烈建议设置 `cwd` 为此项目的根目录，以确保脚本能正确找到其依赖文件（如 `cursor_direct_input.py`）。

2.  将以下自定义规则添加到您的AI助手中 (例如，在 Cursor 的设置 -> Rules -> User Rules):

    ```text
    If requirements or instructions are unclear use the tool interactive_feedback to ask clarifying questions to the user before proceeding, do not make assumptions. Whenever possible, present the user with predefined options through the interactive_feedback MCP tool to facilitate quick decisions.

    Whenever you're about to complete a user request, call the interactive_feedback tool to request user feedback before ending the process. If the feedback is empty you can end the request and don't call the tool in loop.
    ```

    这将确保您的AI助手在提示不明确时以及在标记任务完成之前，总是使用此MCP服务器请求用户反馈。

## 📝 使用技巧

### 处理图片
- **粘贴：** 在反馈窗口的文本输入框中按 `Ctrl+V` (或 `Cmd+V`) 粘贴图片。您可以同时粘贴多张图片和文本。
- **直接对话模式：**
    - 当您添加图片后，提交按钮的文本和颜色会改变，提示您将进入"直接对话模式"。
    - 点击提交后，MCP窗口将关闭，Cursor的聊天输入框会自动激活。
    - 您的文本和所有图片将按顺序自动插入到Cursor聊天框中，并自动发送。
- **键盘快捷键：** 系统使用 `Ctrl+L` (或 `Cmd+L`) 快捷键来激活Cursor的聊天输入框。请确保此快捷键在Cursor中未被更改或冲突。
- **图片预览：** 粘贴的图片会在输入框下方显示可点击的缩略图预览。点击缩略图可以移除对应的图片。

### 常用语
- 点击反馈窗口左下角的"常用语"按钮，可以管理和选择预设的反馈短语，快速填写输入框。

## 🙏 致谢

- 原始概念和初步开发由 Fábio Ferreira ([@fabiomlferreira](https://x.com/fabiomlferreira)) 完成。
- 由 Pau Oliva ([@pof](https://x.com/pof)) 进行了功能增强，并借鉴了 Tommy Tong 的 [interactive-mcp](https://github.com/ttommyth/interactive-mcp) 项目中的一些想法。
- 当前版本由 pawaovo 维护和进一步开发。

## 📄 许可证

此项目使用 MIT 许可证。详情请参阅 `LICENSE` 文件。