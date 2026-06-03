
Chat-to-Codebase 协议说明

Chat-to-Codebase 使用 AUTO_SYNC 文本协议连接网页 AI 与本地 Bridge Server。

它的目标是：

让网页 AI 负责思考和生成代码

让本地脚本负责接收、写入、创建目录、删除、备份和提交

避免复制粘贴多个文件时出错

让聊天中的代码生成结果安全落地到本地代码库

名称关系
项目名：Chat-to-Codebase
协议名：AUTO_SYNC
本地服务：AUTO_SYNC Bridge Server

项目名负责表达整体工作流：从聊天到代码库。
协议名负责标识网页 AI 输出的同步块。

权限边界

假设项目根目录是：

D:/project/test

那么 Chat-to-Codebase 的权限边界是：

D:/project/test/ 内部：允许控制
D:/project/test/ 外部：禁止控制

也就是说，FILE 后面的路径必须永远是相对项目根目录的路径。

允许：

index.html
src/main.js
clients/chatgpt/start_prompt.md
old_folder/

禁止：

../other_project/file.js
D:/project/other_project/file.js
C:/Users/name/Desktop/file.txt
/root/file.txt
~/file.txt

这些被禁止的路径不是因为路径本身特殊，而是因为它们越过了项目根目录边界。

受保护路径

默认情况下，以下路径即使位于项目根目录内，也会被保护：

.git
.env
.env.*
私钥文件
证书文件

其中 .env.example 允许操作。

如果确实需要解除保护，可以在 .env 中设置：

AUTO_SYNC_ALLOW_PROTECTED_PATHS=true

不建议日常开启。

基本结构

为了避免本文档被同步器误判，下面使用占位符描述：

START_MARK
ACTION: write
FILE: path/to/file.ext
完整文件内容
END_MARK

实际输出时：

START_MARK 表示 AUTO_SYNC 开始标记

END_MARK 表示 AUTO_SYNC 结束标记

写入文件

默认动作就是 write，因此 ACTION 可以省略。

START_MARK
FILE: index.html
完整 HTML 内容
END_MARK

也可以显式写：

START_MARK
ACTION: write
FILE: index.html
完整 HTML 内容
END_MARK
创建目录
START_MARK
ACTION: mkdir
FILE: src/components/
END_MARK

如果目录已经存在，则不会重复创建。

删除文件或目录
START_MARK
ACTION: delete
FILE: old_file.py
END_MARK

删除目录：

START_MARK
ACTION: delete
FILE: old_folder/
END_MARK

删除前会根据配置自动备份。

多文件同步

一个文件一个同步块。

START_MARK
FILE: index.html
完整内容
END_MARK

START_MARK
FILE: style.css
完整内容
END_MARK
路径规则

FILE 后面必须是相对项目根目录的路径。

允许：

index.html
src/main.js
clients/chatgpt/tampermonkey.user.js

禁止：

C:/Users/name/project/file.txt
/root/file.txt
../file.txt
.env
.git/config
安全限制

本地接收器会拦截：

绝对路径

Windows 盘符路径

上级目录路径

用户目录路径

.git

.env

.env.*，但允许 .env.example

SSH 私钥文件

证书和密钥文件

输出建议

网页 AI 输出同步块时，建议把所有同步块包在 Markdown text 代码块里。

原因是：Python、YAML、Markdown 等文件对缩进敏感，直接裸输出可能被网页渲染破坏缩进。

协作规则

如果用户只是讨论、分析、设计，不要输出同步块。

当用户明确表示：

写入项目

直接改进去

落地

按这个方案改

可以，直接写入项目

才输出同步块。

涉及删除、密钥、自动 push、核心配置时，建议先提醒用户风险。