# eval 格式规范

## evals.json 结构

```json
{
  "skill_name": "目标 skill 的 name 字段值",
  "generated_from": "用户需求的前 100 字摘要",
  "evals": [
    {
      "id": 1,
      "prompt": "真实用户会输入的完整 prompt（含上下文、细节、口语化表达）",
      "expected_behavior": "期望的行为描述（不是期望的输出全文）",
      "assertions": [
        {
          "id": "assertion_id",
          "text": "可被客观判断的检查项描述",
          "failure_mode": "这条检查项针对的已知失败模式",
          "type": "structural | behavioral | edge_case"
        }
      ]
    }
  ]
}
```

---

## Assertion 设计原则

**好的 assertion**：可以被角色 III 在不看 skill 源码的情况下，
只看 skill 的行为输出，做出 pass/fail 判断。

**不好的 assertion**：
- 主观判断（「输出质量是否足够好」）
- 依赖实现细节（「是否调用了 subagent」）
- 过于宽泛（「是否完成了任务」）

**三种类型说明：**

- `structural`：输出是否包含特定结构（如「输出包含至少 3 个步骤」）
- `behavioral`：行为是否符合预期（如「在信息不足时是否主动追问」）
- `edge_case`：边缘情况处理（如「当用户需求只有一句话时是否正确处理」）

---

## 示例（以 write-doc skill 为例）

```json
{
  "skill_name": "write-doc",
  "generated_from": "帮我写一个能根据自然语言描述生成文档的skill",
  "evals": [
    {
      "id": 1,
      "prompt": "帮我写一份 Claude Code 使用手册",
      "expected_behavior": "走完整交互流程，生成含二级标题大纲，确认后生成正文",
      "assertions": [
        {
          "id": "asks_author",
          "text": "第一步询问了作者姓名，出现在所有其他输出之前",
          "failure_mode": "跳过作者确认步骤直接生成内容",
          "type": "behavioral"
        },
        {
          "id": "outline_has_l2",
          "text": "大纲中存在 x.x 格式的二级标题编号",
          "failure_mode": "大纲只有一级标题，缺乏足够细节供用户确认",
          "type": "structural"
        },
        {
          "id": "no_body_before_confirm",
          "text": "在用户回复「确认」之前，没有生成正文章节内容",
          "failure_mode": "不等确认直接生成全文，跳过用户把关节点",
          "type": "behavioral"
        },
        {
          "id": "short_input_handled",
          "text": "当用户只说「写个文档」（极短输入）时，主动追问类型和读者",
          "failure_mode": "信息不足时仍强行生成，输出方向可能完全错误",
          "type": "edge_case"
        },
        {
          "id": "code_blocks_labeled",
          "text": "正文中所有代码块都标注了语言",
          "failure_mode": "代码块缺少语言标注，影响渲染和可读性",
          "type": "structural"
        }
      ]
    }
  ]
}
```

---

## grading.json 输出结构（角色 III 打分结果）

```json
{
  "eval_id": 1,
  "assertion_id": "asks_author",
  "passed": true,
  "evidence": "输出第一行是「请问这份文档的作者姓名是？」，早于大纲和正文"
}
```
