# 三个角色的上下文边界清单

主流程在调用任何 subagent 前，必须按照此文件的清单精确构造输入。
**清单之外的内容一律不得传入**，这是防止认知污染的硬性约束，不是建议。

---

## 角色 I：需求纯净生成者

**核心职责**：只基于用户的原始需求做事，永不接触 skill 的规范体系或已有实现。
这个隔离是整个流程中最重要的——tmp_claude 的价值来自于它未被规范污染的直觉性，
eval 的价值来自于它不围绕已有实现展开。

### 调用 1：生成 tmp_claude

**允许传入：**
- 用户原始需求描述（逐字）
- 交互追问的完整记录（用户补充的所有内容）
- 任务指令：「按你对这个需求的直觉理解，生成一个 SKILL.md 草稿」

**禁止传入：**
- skill-creator/SKILL.md 的任何内容
- 任何关于"标准 skill 应该长什么样"的描述
- 其他已存在 skill 的示例
- 任何暗示"你应该按规范写"的措辞

**输出**：tmp_claude/SKILL.md

---

### 调用 2：设计 evals（全新实例，与调用 1 完全隔离）

**允许传入：**
- 用户原始需求描述（逐字，与调用 1 相同的原始版本）
- 交互追问的完整记录
- 任务指令：「为这个 skill 设计测试用例和 assertions，
  基于需求规格而不是任何具体实现。
  至少 5 条 assertions，其中至少 2 条专门针对边缘情况和失败模式。
  每条 assertion 注明它针对的是哪种失败模式。」
- eval-schema.md 的格式规范

**禁止传入：**
- tmp_claude/SKILL.md 的任何内容
- tmp_standard/SKILL.md 的任何内容
- 调用 1 的任何输出或推理过程
- skill-creator 的规范内容

**输出**：evals/evals.json

---

## 角色 II：标准构建者

**核心职责**：深度理解 skill-creator 规范，每次调用都以干净的状态接收
"当前版本 + 明确的改进指令"，输出符合规范的 skill。

### 调用 1：生成 tmp_standard（首次）

**允许传入：**
- 用户原始需求描述（逐字）
- 交互追问的完整记录
- skill-creator/SKILL.md 全文
- 任务指令：「按 skill-creator 规范，从零开始实现这个 skill」

**禁止传入：**
- tmp_claude/SKILL.md 的任何内容（防止锚定效应）
- 盲评对比结果（此时尚未执行）
- 任何"另一个版本是这样写的"的暗示

**输出**：tmp_standard/SKILL.md

---

### 调用 N：迭代改进（每轮全新实例）

每轮迭代都是全新实例，不携带上一轮的推理过程。
这样做的原因：防止历史设计决策锚定改进方向，让每轮改进都基于当前材料做独立判断。

**允许传入：**
- 当前 tmp_standard/SKILL.md 全文
- 本轮失败的 assertions 列表（含 evidence）
- 对比分析摘要（提炼后的，非原始推理）
- 用户反馈（如有，优先级最高）
- 改进指令：「针对以上问题改进这个 skill，
  重点关注失败的 assertions 和用户明确指出的问题」
- skill-creator/SKILL.md 全文

**禁止传入：**
- 前 N-1 轮的完整推理过程
- tmp_claude/SKILL.md（每轮都不传）
- 通过的 assertions 的详细 evidence（只需知道哪些通过了即可）

**输出**：改进后的 tmp_standard/SKILL.md

---

## 角色 III：独立评估者

**核心职责**：只做评估，永不生成 skill 内容。
这个约束不能例外——评估者一旦开始"建议怎么改"，它的评估客观性就会打折。

### 调用类型 C：盲评对比

**允许传入：**
- 版本 X 全文（tmp_claude 或 tmp_standard，由主流程随机分配，不标注身份）
- 版本 Y 全文（另一个版本，同样不标注身份）
- 评估维度清单（见 SKILL.md 第三步）
- 任务指令：「对比两个版本，输出结构化差异摘要。
  特别列出：版本 X 有但版本 Y 没有的内容；版本 Y 有但版本 X 没有的内容。
  不要建议如何修改，只做描述性分析。」

**禁止传入：**
- 任何关于"哪个版本是谁生成的"的信息
- 任何生成这两个版本的推理过程
- 用户需求原文（防止评估者倾向于"更接近需求描述"的版本）

**输出**：结构化差异摘要（主流程去盲后解读）

---

### 调用类型 E：assertion 打分（每条独立实例）

每条 assertion、每个测试用例都是独立的角色 III 实例，互相完全隔离。
原因：连续评估多个用例会产生"基准感"，后续评分会受前面结果影响。

**允许传入（单次调用，单条 assertion）：**
- 单个测试用例的输入 prompt
- skill 对该 prompt 的输出
- 单条 assertion 的描述
- 打分指令：「判断这条 assertion 是否通过，输出 pass/fail 和 evidence」

**禁止传入：**
- 其他测试用例的结果
- 其他 assertion 的评分
- 任何关于"整体通过率"的信息
- skill 的源代码（只看行为输出，不看实现）

**输出**：`{ "passed": true/false, "evidence": "具体说明" }`

---

## 主流程的信息提炼责任

主流程在各角色之间传递的是"提炼后的信息"，不是原始输出的全文转发。
提炼时使用结构化格式（JSON 或明确分节的 Markdown），不用自然语言摘要，
以减少主流程自身引入的解读偏差。

**提炼示例（对比摘要 → 改进指令）：**

```json
{
  "deleted_in_standard": [
    "tmp_claude 中的「用户确认节点」描述，tmp_standard 未包含"
  ],
  "added_in_standard": [
    "frontmatter description 更详细的触发条件描述"
  ],
  "failed_assertions": [
    {
      "assertion": "输出包含至少 3 个步骤",
      "evidence": "当前输出只有 2 个步骤"
    }
  ],
  "user_feedback": "希望步骤描述更具体，要有示例命令",
  "improvement_priority": "user_feedback > failed_assertions > deleted_content"
}
```
