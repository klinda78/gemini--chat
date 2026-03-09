❗补充说明

文档里有一项：

Native skill commands can be exposed as slash commands; unauthorized senders see them as plain text.

这意味着：

如果你有一个 skill 定义得足够标准

OpenClaw 会在支持的渠道上注册它作为 slash command

用户可以通过 /skillName 在聊天界面执行它

但这依然是 聊天触发的 skill 执行，而不是 workflow 步骤定义里的 command:

📌 简单总结

✔ Slash Command 是聊天层命令，用于即时执行操作。
❌ Slash Command 不能直接放到 lobster workflow 的步骤里。
✔ Workflow 步骤只能调用实际的 CLI / script / tool
