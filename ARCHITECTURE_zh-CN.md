# Pattern 8 (P8) 体系架构与源码导读

这份文档专为想要深度阅读 P8 (Pattern 8) 源码、了解底层工作机制、或者打算向 P8 贡献代码的中国开发者准备。

由于 P8 致力于成为一个国际化的开源框架，我们**所有的 Python 源代码和注释均强制使用纯英文**。但为了消除母语开发者的阅读屏障，这篇《源码导读》用大白话彻底讲透了它背后的所有逻辑。读完这篇文档，你再看源码绝对会如履平地。

---

## 🏗️ 核心架构思想：法律 vs 警察

P8 的底层设计哲学非常简单，那就是分离**“法律（Law）”**与**“警察（Police）”**。

- **法律 (SKILL)**：也就是暴露在外面给用户手写的 Markdown 和 YAML。业务方根据需求随便改，因为他们最懂业务。
- **警察 (P8 Enforcement Engine)**：藏在 Python 包里的全自动拦截和审计核心。它是冷酷无情的，它只管死磕 YAML 里的规则。Agent 一旦越线，直接被引擎拦下或者踹回去重写。

通过这套架构，AI Agent 从一个“脱缰的野马代码生成器”，变成了一个“带着镣铐跳舞的流水线工人”。

---

## 📂 核心代码目录结构 (`src/p8/`)

```text
src/p8/
├── cli.py                    # 终端命令行入口 (p8 init/list/validate 等)
└── enforcement/              # 警察部门 (MCP Server 拦截引擎)
    ├── mcp_server.py         # 门面：负责和 Agent 沟通的 MCP 协议接口
    ├── reviewer.py           # 文职警察：负责死抠 Agent 生成的文本/代码
    └── security_guard.py     # 武警：负责直接在系统底层拦截危险的 bash 命令
```

---

## 🔍 三大核心 Python 模块解析

接下来，我们将逐一剖析 `enforcement/` 目录下的三个警察。

### 1. `mcp_server.py` — 对外协议大门

**所在文件**：`src/p8/enforcement/mcp_server.py`

由于各大主流 IDE（如 Cursor, Windsurf）和独立 Agent（如 Claude Desktop）都跑在各自的环境里，P8 需要一种标准化的方式去辖制它们。我们选择了 **MCP (Model Context Protocol)**。

这个文件启动了一个 MCP stdio 服务器，向 Agent 暴露了 **3 个只读资源 (Resources)** 和 **2 个强制安检点 (Tools)**。

**3 个 Resources (启动时自动给 Agent 灌输认知)**：
- `skill://index`：告诉 Agent 当前项目有哪些 SKILL 可以用。
- `skill://{name}/checklist`：把工作清单喂给 Agent，让它知道开工前必问哪些问题。
- `skill://{name}/template`：把输出模板喂给 Agent，强制它知道产出物长什么样。

**2 个 Tools (Agent 干活时的强制安检通道)**：
所有的 Agent 在写代码或运行终端前，都只能调用这两个工具。
- `execute_tool(action, skill)`：想在终端敲命令？比如 `npm install` 或者 `rm -rf`？必须把命令先发给这个 Tool。里面会转交给 `SecurityGuard` 审批。
- `submit_review(content, skill)`：写完代码/文档了？想交付？必须把整个文本发给这个 Tool。里面会转交给 `Reviewer` 获取打分。

---

### 2. `security_guard.py` — 操作系统底线的守门人

**所在文件**：`src/p8/enforcement/security_guard.py`

如果在 `skill/references/security.yaml` 里面配了不能干什么（比如严禁删除 `.git`，严禁强推代码，严禁下载不信任的包）。这些规则全是在这里被执行的。

**核心逻辑流程**：
1. `validate_action(action, skill_name)` 这是唯一的主入口。
2. 它会去读取该 SKILL 的 `security.yaml`。
3. 拿着 Agent 想执行的动作（比如字符串 `rm -rf /`）和 YAML 里的正则黑名单（`regex_blacklist`）做逐条比对。
4. 如果命中黑名单，直接返回 `{"allowed": False, "reason": "匹配到黑名单规则: xxx"}`，此时 MCP Server 会直接挂起 Agent 的操作，抛出拒绝错误。

对于 Agent 来说，在操作系统层面被彻底按死了。

---

### 3. `reviewer.py` — 无情的死循环监工

**所在文件**：`src/p8/enforcement/reviewer.py`

这是整个防幻觉机制最核心、最复杂的心脏。当 Agent 觉得自己大功告成，向 `submit_review` 提交它的成果物时，全靠这个 `reviewer.py` 来挑刺。

**核心类：`Reviewer`**
它暴露了一个主方法：`audit(content: str, guidelines: dict, template_content: str)`。

**审计流水线（审核逻辑）**：
1. **强制格式比对 (`_check_format`)**：它会把用户定义的 `template.yaml` 抽出来，看看里面的 Markdown 标题结构是不是在 Agent 的提交物中**原封不动地全部存在**。少一个标题，直接打回。
2. **规则逐条匹配**：它遍历 `guidelines.yaml` 里的规则（rules）。支持以下几种严格约束：
   - `regex_match`: 必须匹配到某个正则（比如必须在开头写版权声明）。
   - `regex_exclude`: 绝对不能匹配到某个正则（比如代码里出现了 `console.log` 测试打印）。
   - `contains`: 必须包含某个具体的字符串。
   - `length_limit`: 审查 Agent 的逼逼赖赖（字数限制），太啰嗦或者太空洞都被拒绝。

**打回机制 (P8AuditError)**：
如果任何一条没过，`Reviewer` 不会只是警告，它会**极其暴躁地**抛出一个名为 `P8AuditError` 的异常。
当异常被扔给 Agent 时，Agent 的 MCP 客户端会看到一个醒目的 **`P8_AUDIT_FAILED`**，以及所有未通过的详细扣分原因。这会触发 Agent 自身的内置纠错机制（Self-Correction），Agent 会乖乖地针对性进行修改并重新提交，直到 P8 满意放行为止。

---

## 🛠️ CLI 工具与多语言支持

最后说一下 `src/p8/cli.py`。
这只是个非常薄的命令行包装壳，用了标准的 `click` 库。
- `p8 init`：直接从库自带的 `scaffold/`（脚手架目录）把标准的 5 大 SKILL 复制到用户的代码库里。
  - **多语言实现**：当你带上 `--lang zh` 参数时，它其实只是把内部存放的一套 `skills_zh/` 复制给了你。
- `p8 list` 和 `p8 validate` 是给开发者调试自己写的 YAML 规则有没有拼写错误的。

---

## 📈 如何参与中文化的建设？

P8 处于早期阶段，如果你想给这个开源库添砖加瓦，你可以：
1. 提交牛逼的中文版 SKILL（提 PR 前请把你的中文精华也翻译到英文版里保持同步）。
2. 在 `reviewer.py` 增加更多智能化的审计维度（比如不仅能正则，甚至能调另一个大模型来当裁判）。
3. 增加对更多框架（比如 React/Vue）的特制预设模板。

读完这篇导游文，你可以直接去翻那三个核心 Python 文件了，你会发现它们短小精悍，却又坚不可摧！
