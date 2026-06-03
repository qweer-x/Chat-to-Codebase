# AUTO_SYNC Bridge

一个轻量级的“网页 AI → 本地项目文件 → Git 提交”自动同步桥。

这个项目的核心想法很简单：

网页大模型负责思考和生成代码。  
本地脚本负责接收、写入、备份、提交。

它不是本地 AI Agent，也不接 Codex。  
它只是把网页 AI 的输出变成可以落地到本地项目的文件同步流程。

## 项目结构

当前推荐结构：

    bridge_server.py
    chatgpt_tampermonkey_script.js
    ai_prompt_manifest.txt
    requirements.txt
    .env.example
    .gitignore
    README.md

旧版文件说明：

    GitHub.py              旧版接收脚本，已被 bridge_server.py 替代
    test_connection.txt    早期测试文件，可以删除

## 工作流程

整体流程：

    你在 ChatGPT 网页里提出需求
        ↓
    ChatGPT 按 AUTO_SYNC 协议输出文件内容
        ↓
    Tampermonkey 脚本捕捉同步块
        ↓
    请求本机 http://127.0.0.1:9999/sync
        ↓
    bridge_server.py 解析文件路径和内容
        ↓
    写入本地项目
        ↓
    自动备份旧文件
        ↓
    可选执行 git add / commit / push

## 安装依赖

建议在项目目录下执行：

    pip install -r requirements.txt

依赖很少，目前只需要 Flask。

## 启动本地接收器

在项目根目录执行：

    python bridge_server.py

启动后访问：

    http://127.0.0.1:9999

如果看到 AUTO_SYNC Bridge Server 状态页，说明本地接收器已经启动成功。

健康检查地址：

    http://127.0.0.1:9999/health

同步接口地址：

    http://127.0.0.1:9999/sync

## 配置方式

可以复制配置模板：

    copy .env.example .env

然后按需修改 .env。

默认配置：

    AUTO_SYNC_PROJECT_ROOT=
    AUTO_SYNC_HOST=127.0.0.1
    AUTO_SYNC_PORT=9999
    AUTO_SYNC_GIT_ENABLED=false
    AUTO_SYNC_GIT_PUSH=false
    AUTO_SYNC_BACKUP_ENABLED=true
    AUTO_SYNC_LOG_DIR=.auto_sync_logs

说明：

- AUTO_SYNC_PROJECT_ROOT 留空时，默认使用启动 bridge_server.py 时所在目录。
- AUTO_SYNC_GIT_ENABLED 为 true 时，写入文件后自动 git add 和 git commit。
- AUTO_SYNC_GIT_PUSH 为 true 时，commit 成功后自动 git push。
- AUTO_SYNC_BACKUP_ENABLED 为 true 时，覆盖旧文件前会备份到日志目录。
- .env 不要上传到 GitHub。

## 安装 ChatGPT 油猴脚本

1. 浏览器安装 Tampermonkey。
2. 新建脚本。
3. 把 chatgpt_tampermonkey_script.js 的内容复制进去。
4. 保存并启用脚本。
5. 打开 chatgpt.com。
6. 页面右下角看到 ChatGPT AUTO_SYNC 面板，即表示脚本已启动。

面板按钮说明：

- 暂停 / 启用：控制是否自动扫描最新回复。
- 重扫最新：手动重新扫描最新一条助手回复。
- 清缓存：清除已发送同步块缓存，方便重复测试。

## AUTO_SYNC 协议

协议由开始标记、FILE 行、完整文件内容、结束标记组成。

为了避免 README 被同步器误判，这里不直接写出完整标记。  
实际使用时，开始标记是 AUTO 与 _SYNC 合并后的标记。  
结束标记是 END 与 _SYNC 合并后的标记。

结构如下：

    开始标记
    FILE: path/to/file.ext
    文件完整内容
    结束标记

注意：

- 一个文件一个同步块。
- FILE 必须是相对项目根目录的路径。
- 不要使用绝对路径。
- 不要使用 .. 路径。
- 不要写入 .env、.git、私钥、证书、Token、Cookie、密码。
- 代码必须完整，不要省略。
- 对缩进敏感的代码建议放在代码块里输出，避免网页渲染破坏缩进。

## 安全限制

bridge_server.py 默认拦截以下路径：

- .git
- .env
- .env.*，但允许 .env.example
- id_rsa、id_ed25519 等 SSH 私钥文件
- .pem、.key、.p12、.pfx、.crt 等证书或密钥文件
- 绝对路径
- Windows 盘符路径
- 上级目录路径

## 日志和备份

默认日志目录：

    .auto_sync_logs

其中：

    .auto_sync_logs/sync.log.jsonl
    .auto_sync_logs/backups/

这些运行时文件已经被 .gitignore 忽略，不建议上传到 GitHub。

## Git 自动提交

默认不自动提交：

    AUTO_SYNC_GIT_ENABLED=false
    AUTO_SYNC_GIT_PUSH=false

如果你确认流程稳定，可以在 .env 中打开：

    AUTO_SYNC_GIT_ENABLED=true
    AUTO_SYNC_GIT_PUSH=true

建议先只打开自动 commit，不要马上打开自动 push。

更稳妥的流程是：

    AUTO_SYNC_GIT_ENABLED=true
    AUTO_SYNC_GIT_PUSH=false

确认本地 commit 没问题后，再手动 push。

## 推荐协作方式

你不需要机械地说“开始同步”或“只讨论”。

自然表达即可：

- “先聊聊这个怎么设计”
- “这个先别写”
- “你觉得怎么改更好”
- “这个可以落地”
- “直接写进项目”
- “按你建议更新一版”

网页 AI 应该像合作伙伴一样参与判断。  
但只要输出同步块，本地脚本就可能立刻写入文件，所以涉及覆盖、删除、密钥、自动推送时应该先确认。

## 旧版迁移

旧版 GitHub.py 已经被 bridge_server.py 替代。

建议迁移完成后删除：

    GitHub.py
    test_connection.txt

然后保留：

    bridge_server.py
    chatgpt_tampermonkey_script.js
    ai_prompt_manifest.txt
    requirements.txt
    .env.example
    .gitignore
    README.md