# CodeGraph 当前三个核心问题：用 Django 查询讲清楚

## 1. 这份报告要说明什么

CodeGraph 的目标不是简单地“搜索代码”，而是帮助 Codex、Claude Code 等 Agent 快速获得
完成任务所需的结构化代码证据，例如：

- 某个请求经过哪些函数；
- 某个方法由谁调用；
- 一次修改可能影响哪些代码；
- 一个入口最终如何到达目标实现。

目前 CodeGraph 已经能在大型 Django 项目中找到大量符号、调用关系和源码，但在真实 Agent
使用场景下仍有三个紧密关联的问题：

1. **返回很多内容，但不一定都是回答问题最需要的内容；**
2. **用户用自然语言或不准确的符号名提问时，检索效果明显下降；**
3. **已经有足够证据后，系统仍可能重复查询、重复返回。**

可以把这三个问题类比成请一位图书管理员帮忙：

| 问题 | 图书管理员的表现 | CodeGraph 中的对应表现 |
|---|---|---|
| 内容分配 | 搬来一整箱书，但真正有用的只有几页 | 返回约 20K～25K 字符，但关键代码只占一部分 |
| 查询理解 | 必须准确说出书名，描述主题或说错一个字就难以找到 | 精确符号查询很好，自然语言和拼写错误召回较差 |
| 充分即停 | 已经找到答案，还继续搬来同样的书 | Agent 继续 explore/Read/Grep，重复查询再次返回完整内容 |

下面使用 Django 源码中的六个简单查询，逐一说明这些问题。

六个查询在 CodeGraph 1.6.0 上的完整原始结果已经逐例保存，可从
[Django 查询示例与完整结果](./query-examples/codegraph-1.6.0/README.md)进入。每个示例一个
文件，包含查询、复现命令、指标和未经删减的 MCP 返回，便于与后续版本逐项比较。

---

## 2. 问题一：返回内容很多，但有效信息比例不够高

### 2.1 问题本质

CodeGraph 当前倾向于在一个较大的固定预算中尽量多提供源码。这样做可以降低遗漏风险，
但“返回得多”不等于“回答得好”。

Agent 真正需要的通常是：

```text
入口函数
→ 关键中间调用
→ 核心分支
→ 最终目标
```

如果响应里混入大量相邻方法、重复定义或与主路径关系不大的代码，会产生两个后果：

- CodeGraph 的一次工具结果占用较多上下文；
- Agent 仍可能认为关键证据不够清楚，转而 Read 文件，造成二次消耗。

### 2.2 示例一：Django 如何调用匹配到的 view

#### 查询方式

直接查询 CodeGraph：

```powershell
codegraph explore "BaseHandler.get_response _get_response resolve_request URLResolver.resolve ResolverMatch callback" `
  --path D:\工作空间\harbor\exp\django
```

也可以这样问 Codex 或 Claude Code：

```text
请解释 Django 从 BaseHandler.get_response 开始，经过 URL 解析，
最终调用匹配到的 view callback 的完整流程。
```

#### 真正需要的答案

这个问题的核心证据很集中：

```text
BaseHandler.get_response
→ BaseHandler._get_response
→ BaseHandler.resolve_request
→ URLResolver.resolve
→ ResolverMatch
→ resolver_match.func / callback
```

关键文件主要是：

```text
django/core/handlers/base.py
django/urls/resolvers.py
```

#### 当前表现

在 Django 测试仓库上，精确符号查询能够覆盖上述关键事实，但一次响应约为：

```text
24,944 字符
约 8,315 tokens（按 3 字符/token 粗略换算）
```

问题不是“答案错误”，而是为了表达一条不算很长的主路径，响应携带了较大的源码包。

#### 为什么这是优化机会

如果系统能够识别出这是一个“入口到 callback 的调用链问题”，就可以优先提供：

1. 完整调用链；
2. 每一跳对应的关键调用语句；
3. URL 匹配成功和失败的核心分支；
4. 必要的方法体片段。

其他邻近方法可以先只给签名或位置。Agent 真有后续问题时，再增量展开。

理想结果不是简单把 24K 截成 10K，而是用更少内容仍然完整回答主路径。

### 2.3 示例二：`Model.save()` 如何决定 update 还是 insert

#### 查询方式

```powershell
codegraph explore "Model.save save_base _save_table _do_update _do_insert" `
  --path D:\工作空间\harbor\exp\django
```

自然语言提问：

```text
Django 的 Model.save 最终在哪里判断执行 UPDATE 还是 INSERT？
请给出最核心的方法调用顺序。
```

#### 真正需要的答案

```text
Model.save
→ Model.save_base
→ Model._save_table
   ├─ Model._do_update
   └─ Model._do_insert
```

关键代码基本集中在：

```text
django/db/models/base.py
```

#### 当前表现

测试中，该查询的关键事实覆盖率为 100%，但仍返回约：

```text
24,862 字符
约 8,287 tokens
```

`base.py` 是一个大型核心文件。按文件或普通相关性分配预算时，很容易把大量预算花在
“与 Model 有关，但不是本问题主路径”的方法上。

#### 理想表现

CodeGraph 应把预算分配到 `_save_table()` 中真正决定 update/insert 的分支，并确保
`_do_update()`、`_do_insert()` 的关键实现被保留。其余内容可以压缩为：

```text
方法签名 + 位置 + 与主路径的关系
```

这个例子说明，优化单位应从“文件”进一步细化到“AST 方法、分支和代码片段”。

### 2.4 这个问题应如何衡量

至少同时观察三个指标：

```text
事实覆盖率：关键入口、出口、分支是否仍然齐全
返回字符/token：为得到这些事实付出了多少上下文
后续回退：Agent 是否仍然使用 Read/Grep
```

只有满足下面的条件，才能算真正优化：

```text
事实覆盖率不下降
并且
返回内容减少，或者 Agent 的 Read/Grep 减少
```

只减少字符而丢失关键代码，是截断，不是优化。

---

## 3. 问题二：自然语言和不准确查询的效果较差

### 3.1 问题本质

CodeGraph 对下面这种查询比较擅长：

```text
BaseHandler._get_response resolve_request URLResolver.resolve ResolverMatch
```

因为输入中已经包含源码里的准确符号名。

但真实用户往往先知道“业务行为”，并不知道实现它的函数名。他们更可能这样问：

```text
Django 收到请求以后，是怎么找到并调用 view 的？
```

这就产生了输入错位：

```text
用户掌握的是概念和行为
CodeGraph 最擅长的是符号和关系
```

### 3.2 示例一：完全使用自然语言描述请求流程

#### 查询方式

```powershell
codegraph explore "how Django takes an incoming request resolves its URL and calls the selected Python view" `
  --path D:\工作空间\harbor\exp\django
```

中文表达可以是：

```text
Django 怎样接收一个请求、匹配 URL，然后调用选中的 Python view？
```

#### 期望找到的符号

即使用户没有说出符号名，系统也应该逐渐关联到：

```text
get_response
_get_response
resolve_request
URLResolver.resolve
ResolverMatch
callback
```

#### 当前表现

在直接 MCP 测试中，该自然语言查询仍返回约 24,996 字符，但关键事实覆盖率只有：

```text
42.9%
```

这是非常有代表性的现象：**输出并不少，但答案需要的符号没有被完整召回。**

它说明单纯扩大返回预算无法解决查询理解问题。候选符号没有找对，后面的排序和预算再大
也只能返回更多不够相关的内容。

#### 理想查询规划过程

系统可以先把自然语言转成一个小型查询计划：

```text
行为：接收请求、解析 URL、调用 view
候选概念：request、resolver、callback、handler
候选符号：get_response、resolve、ResolverMatch
目标关系：handler → resolver → callback
```

然后使用候选符号召回节点，再利用图连通性找出属于同一条路径的组合。

### 3.3 示例二：符号名拼错并混入噪声

#### 查询方式

```powershell
codegraph explore "request response path get_respnse middleware URL matching callback" `
  --path D:\工作空间\harbor\exp\django
```

这里故意把：

```text
get_response
```

写成了：

```text
get_respnse
```

同时混入 `request`、`path`、`middleware` 等宽泛词。

#### 当前表现

测试中，该查询返回约 24,090 字符，但关键事实覆盖率只有：

```text
20%
```

对 Agent 来说，接下来通常只能：

```text
再次换词调用 CodeGraph
或使用 Grep 搜索 get_response
或 Read handler 文件
```

这不仅增加工具调用和 token，也削弱了 Agent 对 CodeGraph 的信任。连续一两次查询效果不好后，
Agent 可能在后续任务中直接放弃 CodeGraph。

#### 理想表现

系统应识别：

```text
get_respnse
与
get_response
字符结构高度相似
```

但不能仅凭编辑距离直接认定答案。更稳妥的过程是：

```text
拼写候选：get_response
上下文验证：它是否与 middleware、URL、callback 有图关系？
结果：有，因此提升 get_response 及其调用邻居的排名
```

这样可以避免把一个拼写近似、但语义无关的符号错误地排到第一位。

### 3.4 为什么普通全文搜索不够

自然语言问题不能只依靠一种搜索算法。更合理的是多路召回：

```text
精确符号匹配
+ 标识符分词
+ trigram / 编辑距离纠错
+ 框架概念映射
+ 图连通性重排
```

例如 `selected Python view` 在源码里未必以完整短语出现，但 `ResolverMatch.func`、
`callback` 和 `_get_response` 之间存在实际结构关系。图关系应该用来验证文本搜索产生的候选。

### 3.5 这个问题应如何衡量

建议准备三组等价查询：

```text
A. 精确符号：BaseHandler._get_response URLResolver.resolve ResolverMatch
B. 自然语言：Django 怎样解析 URL 并调用 view
C. 噪声查询：get_respnse middleware URL matching callback
```

然后比较：

- 三种查询的关键事实覆盖率差距；
- Agent 为完成任务调用 CodeGraph 的次数；
- 是否需要 Grep/Read 才能补齐答案；
- 第一次 MCP 查询是否已经找到正确入口。

优化目标不是让三种查询返回完全相同的文字，而是让它们都能找到同一条正确主路径。

---

## 4. 问题三：已经有足够证据，但系统不会及时停止

### 4.1 问题本质

CodeGraph 当前主要按照“还能返回多少内容”管理查询，而不是按照“问题是否已经得到回答”
管理查询。

这两个判断并不相同：

```text
还有预算 ≠ 还需要继续查
没有超过调用上限 ≠ 新一次调用有价值
```

真正的停止条件应该来自证据状态，例如：

```text
入口是否找到？
出口是否找到？
入口到出口是否连通？
关键分支是否覆盖？
下一次查询还能增加多少新信息？
```

### 4.2 示例一：一次查询已经覆盖完整 callback 路径

#### 查询方式

```powershell
codegraph explore "BaseHandler._get_response resolve_request URLResolver.resolve ResolverMatch callback" `
  --path D:\工作空间\harbor\exp\django
```

#### 当前测试结果

该查询返回约 20,078 字符，关键事实覆盖率为 100%。从回答问题的角度看，入口、解析器、
匹配结果和 callback 已经齐全。

这时 Agent 的理想行为是：

```text
读取 MCP 结果
→ 组织答案
→ 停止检索
```

不理想行为是：

```text
读取 MCP 结果
→ 再次 explore 相近符号
→ Grep ResolverMatch
→ Read base.py
→ 最后才回答
```

后面几步未必提高答案准确率，却会继续增加耗时和上下文占用。

#### CodeGraph 可以提供什么帮助

停止不能完全依赖 Agent 自觉判断。CodeGraph 可以在结果中提供机器可读的充分性信号：

```text
入口：已覆盖
出口：已覆盖
路径：已连通
关键文件：已返回
未解决动态边界：无
建议：证据已足够，可以直接回答
```

这比简单显示“本项目建议最多调用两次”更有意义。

### 4.3 示例二：完全重复同一个查询

#### 查询方式

在同一个 MCP 会话中连续调用两次：

```text
Model.save save_base _save_table _do_update _do_insert
```

#### 当前测试结果

两次调用分别返回：

```text
第一次：24,862 字符
第二次：24,862 字符
第二次 / 第一次：1.0
总计：49,724 字符，约 16,575 tokens
```

第二次查询没有带来新的问题，也没有增加新的符号，但系统仍再次返回完整内容。

这是最容易理解的“不会停止”案例：用户第二次问了几乎相同的问题，系统又把同一箱书
完整搬了一遍。

#### 理想表现

第二次响应可以缩减为：

```text
No new evidence.

Already covered:
Model.save → save_base → _save_table
                            ├─ _do_update
                            └─ _do_insert

Relevant source was returned in the previous call.
```

如果第二次查询增加了 `_prepare_related_fields_for_save`，则只返回新增部分以及它与旧路径的连接：

```text
New evidence:
Model.save → _prepare_related_fields_for_save

Previously covered path omitted.
```

这就是“增量响应”，而不是机械地再次生成完整响应。

### 4.4 需要什么样的会话状态

CodeGraph 可以为每个 MCP 会话维护一个轻量证据集合：

```text
已返回的符号 ID
已返回的代码行区间
已返回的图边
已覆盖的调用路径
查询的规范化表示
```

每次生成新响应前，计算：

```text
新增信息 = 当前候选证据 - 会话中已有证据
```

再计算边际收益：

```text
边际收益 = 新增关键证据数量 / 本次预计 token 数
```

当新增信息接近零，或者边际收益低于阈值时，应停止展开，只返回简短状态。

### 4.5 这个问题应如何衡量

重点观察：

- 一次充分响应后，Agent 是否直接回答；
- 最后一次 CodeGraph 调用后还有多少 Read/Grep；
- 相同查询第二次响应与第一次响应的大小比例；
- 多轮会话累计占用多少输入 token；
- 增量问题是否只返回新增证据。

对于完全重复的查询，可以把下面的目标作为直观起点：

```text
第二次响应字符数 / 第一次响应字符数 <= 0.45
```

更理想的结果应远低于这个值。

---

## 5. 三个问题不是独立的

它们形成了一条连续链路：

```text
用户自然语言提问
        ↓
查询理解不准，候选符号偏离
        ↓
为了避免遗漏，返回更多内容
        ↓
关键证据仍不够突出
        ↓
Agent 继续 explore / Read / Grep
        ↓
工具调用和累计 token 继续增加
```

反过来，完整的优化闭环应该是：

```text
自然语言查询规划
        ↓
精确召回候选符号
        ↓
使用图关系重排候选
        ↓
按信息价值/token 选择代码片段
        ↓
更新会话证据状态
        ↓
证据充分则停止；否则只查询缺口
```

因此，不能只调小输出上限，也不能只添加拼写纠错。三个模块需要共享同一份任务状态。

---

## 6. 可以怎样向别人概括

如果只用一分钟介绍，可以这样说：

> CodeGraph 当前精确符号查询已经比较强，但真实 Agent 不总能给出精确符号。它用自然语言
> 或拼错名称时，CodeGraph 可能返回很多内容却没有覆盖正确路径；即使已经找到答案，重复
> 查询还可能再次返回完整源码。因此下一阶段不只是“搜索更快”，而是让系统理解要找什么、
> 只返回最有价值的证据，并在证据足够时及时停止。

如果只展示三个数字，可以使用本次 Django 直接 MCP 测试结果：

| 现象 | 示例结果 | 说明 |
|---|---:|---|
| 精确查询内容较大 | 约 24.9K 字符 | 事实齐全，但仍有压缩和重分配空间 |
| 自然语言/拼写错误召回下降 | 42.9% / 20% 覆盖率 | 输出没有减少，关键事实却明显缺失 |
| 重复查询没有缩减 | 第二次/第一次 = 1.0 | 同一内容重复进入上下文 |

这些数字来自固定 Django 用例，适合用于优化前后的回归比较，不应被理解为 CodeGraph 在所有
项目上的统一比例。

---

## 7. 分享时建议的演示顺序

建议现场演示按以下顺序进行：

1. 先运行精确的 `Model.save` 查询，说明 CodeGraph 已经能够正确找到主流程；
2. 再运行自然语言的 request/view 查询，展示查询输入变化带来的覆盖率下降；
3. 再运行拼错的 `get_respnse` 查询，让问题变得直观；
4. 最后在同一会话重复 `Model.save` 查询，展示完整响应被重复发送；
5. 用“查询规划 → 信息分配 → 充分即停”总结后续算法方向。

标准测试命令：

```powershell
cd D:\工作空间\harbor\codegraph

# 不调用模型，只跑快速、稳定的 MCP 测试
node scripts/agent-eval/cross-agent-eval.mjs probe `
  --out .agent-eval/sharing-demo

# 查看自动生成的结果表
Get-Content .agent-eval/sharing-demo/report.md
```

需要展示真实 Agent 行为时：

```powershell
node scripts/agent-eval/cross-agent-eval.mjs run `
  --agent codex --repeats 2 --no-build `
  --out .agent-eval/sharing-codex
```

真实 Agent 运行会把任务以及必要的代码片段发送到对应模型服务；只希望本地测试时，应使用
`probe`。
