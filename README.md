# Chat-to-Codebase

Chat-to-Codebase 是一个轻量级的网页 AI 自动同步桥。

它的核心模式是：

```text
网页大模型负责思考和生成代码
本地脚本负责接收、写入、删除、备份和提交

也可以理解为：

从聊天到代码库
Chat → Codebase

它不是本地 AI Agent，也不接 Codex。
它只负责把网页 AI 的代码输出，安全地落地到本地项目代码库中。

工作流程
ChatGPT / 其他网页 AI
        ↓
输出 AUTO_SYNC 同步块
        ↓
Tampermonkey 脚本捕捉
        ↓
发送到 http://127.0.0.1:9999/sync
        ↓
bridge_server.py 写入本地项目
        ↓
可选 Git commit / push
权限边界

Chat-to-Codebase 对项目根目录内部拥有完整控制权：

创建文件
修改文件
覆盖文件
删除文件
创建目录
删除目录
重构目录结构

Chat-to-Codebase 对项目根目录外部没有任何权限。

例如项目根目录是：

D:/project/test

那么它只能操作：

D:/project/test/

下面的内容，不能越过这个目录。

默认情况下，.git、.env、私钥、证书等敏感路径仍会被保护。
如确实需要解除保护，可设置：

AUTO_SYNC_ALLOW_PROTECTED_PATHS=true

不建议日常开启。

项目结构
bridge_server.py
requirements.txt
.env.example
.gitignore
README.md

clients/
└─ chatgpt/
   ├─ tampermonkey.user.js
   └─ start_prompt.md

docs/
└─ protocol.md

以后要支持其他网页模型，可以继续添加：

clients/
├─ chatgpt/
├─ gemini/
├─ claude/
├─ grok/
└─ openrouter/

每个模型目录里放自己的网页脚本和启动提示词。

安装
Bash
pip install -r requirements.txt
启动

在目标项目根目录运行：

Bash
python bridge_server.py

打开：

http://127.0.0.1:9999

看到状态页，说明本地接收器启动成功。

安装 ChatGPT 脚本

安装浏览器插件 Tampermonkey

新建脚本

复制 clients/chatgpt/tampermonkey.user.js 的内容

保存并启用

打开 chatgpt.com

右下角出现 Chat-to-Codebase 面板即成功

新对话使用方式

打开新的 ChatGPT 对话后，先复制：

clients/chatgpt/start_prompt.md

把里面的内容发给 ChatGPT。

之后就可以自然协作：

先设计一下，不要写入。

确认方案后再说：

可以，直接写入项目。

ChatGPT 输出同步块后，脚本会自动发送到本地接收器。

Git 提交

默认只写入文件，不自动提交。

复制配置模板：

Bash
copy .env.example .env

然后修改：

AUTO_SYNC_GIT_ENABLED=true
AUTO_SYNC_GIT_PUSH=false

建议先只开自动 commit，不要马上开自动 push。

支持动作
ACTION: write
ACTION: mkdir
ACTION: delete

默认不写 ACTION 时，按 write 处理。

安全限制

本地接收器会拦截：

项目根目录外路径

绝对路径

Windows 盘符路径

上级目录路径

用户目录路径

.git

.env

私钥文件

证书文件

日志和备份默认保存在：

.auto_sync_logs/

这个目录不会上传到 GitHub。

当前状态

v0.1 已验证通过：

ChatGPT 新对话
  ↓
AUTO_SYNC 同步块
  ↓
Tampermonkey
  ↓
bridge_server.py
  ↓
本地文件
  ↓
GitHub 仓库
仓库命名建议

推荐 GitHub 仓库名：

chat-to-codebase

如果你已经有旧仓库，可以在 GitHub 仓库设置里手动改名，然后本地执行：

Bash
git remote set-url origin https://github.com/qweer-x/chat-to-codebase.git