你正在修改仓库根目录下的 `git-ai-commit` Bash 脚本。

目标：
修复当前 AI 生成 commit message 的两个核心问题。

1. 生成的 commit message 经常不能覆盖全部变更内容。
2. 生成结果缺乏详细描述，通常只有一行 title，没有 body。

请先阅读并分析以下事实，再直接修改代码，而不是只给建议。

已知现状和问题定位：

1. 当前脚本默认使用 `git diff`，也就是分析 unstaged 变更；但真正提交前会执行 `git add -A`，导致“分析范围”和“实际提交范围”不一致。
2. 当前脚本会对原始 `git diff` 直接按行数截断，例如 `head -n "$MAX_DIFF_LINES"`。这会导致：
   - 后面的文件完全丢失
   - 甚至可能从 hunk 中间截断
   - 模型拿到的是不完整 patch
3. 当前 prompt 偏向生成简短标题，而不是完整 commit message。
4. 当前脚本最终会把模型输出裁成第一行，因此即使模型返回了正文也会被丢掉。
5. 当前 prompt 中存在冲突要求，例如“如果涉及多个 feature 就拆成多个 commits”，但工具本身一次只生成一个 commit message。

本次改造要求：

1. 只基于 staged 生成并提交。
   - 默认只分析 staged 内容。
   - 提交时不要再偷偷把 unstaged 内容一起 `git add -A`。
   - 如果没有 staged 变更，应明确报错提示。

2. 不要再对原始 `git diff` 直接 `head`。
   - 至少补充：
     - `git diff --staged --stat`
     - `git diff --staged --name-status`
   - 大 diff 时，优先按文件或变更组做摘要，再汇总给模型。
   - 目标是让模型尽可能覆盖全部主要变更，而不是只看到前 500 行 patch。

3. 把输出格式改成强约束模板。
   - 目标格式：

```text
type(scope): short summary

- summarize change group 1
- summarize change group 2
- summarize change group 3
```

   - 第一行是 conventional commit subject。
   - 后续 body 用 bullet 列出主要改动组。
   - body 要覆盖主要文件范围、行为变化，以及必要时的兼容性/风险、测试、文档变更。

4. 删除冲突提示。
   - 不要再要求“split into multiple commits”。
   - 改成“cover all major change groups in one message”。

5. 不要再把输出裁成一行。
   - 保留多行 commit message。
   - 使用 `git commit -F <tempfile>` 或其他可靠方式提交多行内容。
   - dry-run 模式下也应完整打印 subject + body。

6. 把 prompt 从“concise title”改成“subject + body”。
   - 明确要求模型：
     - 输出 conventional commit subject
     - 输出 body bullet points
     - 覆盖全部主要改动组
     - 不要遗漏重要文件和行为变化
     - 不要输出额外解释性前后缀

实现要求：

1. 优先修改现有脚本，不要引入过重依赖。
2. 保持 Claude / OpenAI / Gemini 三个 provider 的行为一致。
3. 如果需要新增辅助函数，命名清晰，尽量复用。
4. 保持脚本可读性，避免过度复杂。

验收标准：

1. 默认运行时只处理 staged 变更。
2. 对于多文件修改，生成的 commit body 能覆盖多个主要改动组。
3. dry-run 可以看到完整多行 commit message。
4. 实际提交保留多行内容，而不是只保留 title。
5. 不再出现“分析的是 unstaged，提交的是所有改动”这种不一致。

输出要求：

1. 直接修改代码。
2. 完成后说明：
   - 改了哪些核心逻辑
   - 为什么这样改
   - 还有哪些边界情况没有覆盖
