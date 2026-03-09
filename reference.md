$LOBSTER 只是代表命令行工具本身的标识符，在实际使用中，你应该将 $LOBSTER 替换成 OpenClaw 的工具命令行接口。

例如，命令可以是：
``` bash
openclaw lobster '<pipeline>'
```
或者在某些系统中，可能使用特定工具调用：
``` bash
lobster '<pipeline>'
``` 
## 文档中的erfrence 具体含义如下：

**1. Run pipeline (human mode - pretty output)**
``` bash
$LOBSTER '<pipeline>'
```
- 这条命令会运行指定的 pipeline（即一系列自动化步骤），并以易读的格式显示输出，通常适用于人类查看的格式，输出会有 颜色和格式 来帮助理解结果。

**2. Run pipeline (tool mode - JSON envelope for integration)**
``` bash
$LOBSTER run --mode tool '<pipeline>'
```
- 这条命令会以 tool mode 运行 pipeline，并且输出是以 JSON 格式 包装的，适合系统集成或程序化处理，输出没有漂亮的格式，而是用 JSON 数据格式返回，适合开发人员或 API 调用。

**3. Run workflow file**
``` bash
$LOBSTER run path/to/workflow.lobster
```
- 这个命令运行一个具体的 workflow 文件，即执行你提前定义好的 .lobster 文件。path/to/workflow.lobster 是文件的路径。

**4. Resume after approval**
``` bash
$LOBSTER resume --token "<token>" --approve yes|no
```
- 这个命令用于 恢复 一个已经被暂停的流程，通常是在需要 人工审批 的情境下。你提供一个 token（通常是之前操作中生成的标识符），并且选择是否 批准 继续执行：yes 或 no。

**5. List commands/workflows**
``` bash
$LOBSTER commands.list
$LOBSTER workflows.list
```
- 这些命令用于列出当前可用的 命令 或 workflow 文件，帮助你查看当前环境里可以执行的任务或流程。
