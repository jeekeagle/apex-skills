# review-get — Smoke-Test 回归用例

每次修改 `SKILL.md` 之后，跑一遍这里的 6 个用例。任何一条过不去，要么是规则需要调整，要么是输出写错了——**不要改用例来迁就输出**。

## 验证器（11 项硬规则）

每条回复都要过这个 checklist（直接对应 `SKILL.md` 里的"验证清单"段）：

```
1. 第一句话以"你"开头（或以"开/关"中文引号开头后立即跟"你"）
2. 不出现内部词: 素材库 / 时间线 / 四字诀 / 兜底 / Skill 名称
3. 不用 --- 分割线 / ## ### 标题 / 🔥💡🎯⚖️ emoji 小标题
4. 不用表格 / > 引用块 / 1.2.3. 编号列表
5. 没有 AI 腔禁词: 很棒 / 很有价值 / 加油 / 希望对你有帮助 / 你的思考很深刻 / 你的视角很独到 / 值得肯定的是
6. 没有产品/学术/心理学黑话: 复盘 / 闭环 / 接住 / 洞察 / 赋能 / 心智模型 / 底层逻辑
7. 不暴露内部动作: 让我 / 我搜 / 我看了 / 我扫了
8. 字数 ≤ 400 (正常 150-300)
9. 一次只挑 1 个亮点,不同时夸多项
10. 没有转折/暗示不完美: 但是 / 不过 / 如果能再加上
11. 整条消息是连贯的聊天消息
```

外加 2 个内容归属规则：

```
12. 用户原创 → "你"开头自然连贯
13. 他人内容 → 必须用 "你记的" / "你划的" / "你停" 这类, 不能用 "你写的"
```

## 6 个真实 trigger

每个 trigger 后跟一段**期望风格的回复**（不是固定答案——文字可以不同，但风格必须遵守上面的 11+2 项）。

### Trigger 1: 短内容 + 用户原创 + 一句话点破

输入:

> 今天跟客户提案,他们一开始反对,我没急着解释,先重复了一遍他们的担心。后来谈下来感觉还挺顺的。

期望风格: 1-2 段, ≤120 字, 挑一个"先重复了担心"的动作, 指出"对方觉得被听见"为什么是关键。

### Trigger 2: 散内容 + 用户原创 + 聚焦不夸

输入:

> 我觉得沟通最重要的是真诚,要站在对方角度思考,理解对方的需求,然后找到双赢的方案。还有就是要注意措辞,不要太生硬。

期望风格: 1-2 段, ≤150 字, **只点一个"你自己的"经验**(这里是"不要太生硬"), 前面套话全部跳过; 结尾用"多说说"形式自然延伸。

### Trigger 3: 他人内容 + 眼光型

输入:

> 今天读到一段话挺有感觉的:「管理的本质不是控制,是释放。你不需要让每个人都听你的,你需要让每个人都知道自己该干什么。」

期望风格: 1-2 段, ≤150 字, **必须用"你记的这条"或"你划的这句"开头**(他人内容归属), 点出"为什么这条打动了 ta"(眼光)。

### Trigger 4: 自我怀疑情绪 + 先回应情绪再说内容

输入:

> 我上周写的产品反思,老板看完没回我。我不知道是写得不行,还是他太忙了。其实我自己也觉得写得一般,但又不知道哪里可以改。

期望风格: 2 段, ≤200 字, **第一段回应情绪**(不让用户感到被评判), 第二段给出具体的一个动作(让他贴出一句最不满意的)。禁止"你应该 / 建议你"。

### Trigger 5: A 兜底(内容太散)

输入:

> 嗯,听完了,感觉有点意思。

期望风格: 1 段, ≤80 字, **第一句以"你"开头**(不是"我看见了"), 一句"我看见了"对应的具体回应, + 一个具体问题引导用户多说一句。

### Trigger 6: 没附内容兜底

输入:

> 点评一下

期望风格: 1 段, ≤80 字, **第一句以"你"开头**(不是"点评需要原料"或"我看见"), 老实告诉用户把要点评的内容贴过来, 不翻用户的 obsidian / getnote / 任何文件。

## 自动化检查脚本

把这段贴到 `execute_code` 跑一遍：

```python
import re

def check_reply(reply, must_contain=None, must_not_contain=None):
    issues = []
    first = reply.lstrip().split('\n')[0]
    if not (first.startswith('你') or (first.startswith('"') and '你' in first[:5])):
        issues.append(f"first sentence must start with 你: {first[:30]!r}")
    for w in ['素材库', '时间线', '四字诀', '兜底', 'Skill名称']:
        if w in reply: issues.append(f"internal word: {w}")
    for w in ['\n---', '\n##', '\n###', '🔥', '💡', '🎯', '⚖️']:
        if w in reply: issues.append(f"format violation: {w!r}")
    if re.search(r'\n\d+\.', reply): issues.append("numbered list")
    for w in ['很棒', '很有价值', '加油', '希望对你有帮助', '值得肯定的是', '你的思考很深刻', '你的视角很独到']:
        if w in reply: issues.append(f"AI 禁词: {w}")
    for w in ['复盘', '闭环', '接住', '洞察', '赋能', '心智模型', '底层逻辑']:
        if w in reply: issues.append(f"黑话: {w}")
    for w in ['让我', '我搜', '我看了', '我扫']:
        if w in reply: issues.append(f"暴露动作: {w}")
    if len(reply) > 400: issues.append(f"length {len(reply)} > 400")
    for w in ['但是', '不过', '如果能再加上']:
        if w in reply: issues.append(f"转折/暗示不完美: {w}")
    if must_contain:
        for w in must_contain:
            if w not in reply: issues.append(f"missing: {w}")
    if must_not_contain:
        for w in must_not_contain:
            if w in reply: issues.append(f"should not contain: {w}")
    return issues
```

调用示例（以 Trigger 3 为例，他人内容必须用"你记的"）:

```python
reply = """你记的这条,大多数人读到会觉得"有道理"就划走了。你停下来存了——说明你可能正好卡在这个问题上:管还是不管,管多深。

"让每个人都知道自己该干什么"——难不在理解,在做到。你团队里谁是你最拿不准"他到底知不知道自己该干什么"的人?"""
issues = check_reply(reply, must_contain=["你记的"], must_not_contain=["你写的"])
assert not issues, issues
```

## 修订历史

- 2026-06-03: 初版 6 个 trigger,从首次 smoke test 沉淀。Trigger 5(A 兜底)和 Trigger 6(没附内容)曾因"我看见了/点评需要原料"开头违反"以你开头"硬规则而被回炉重写。
