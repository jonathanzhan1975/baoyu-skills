
---
name: wechat-article-pipeline
description: 微信公众号文章全流程生成与发布 SOP (Pipeline)。从选题、搜索、撰写、格式化到保存草稿一站式完成。触发词：「我要写一篇公众号文章」「帮我生成一篇关于XXX的文章」「创建公众号内容」。支持多账号管理，关键节点确认。
---
# 微信公众号文章生成与发布 SOP
一站式完成公众号文章的选题、搜索、撰写、格式化和发布流程。
## 触发方式
**自然语言触发**，当用户说：
- 「我要写一篇公众号文章」
- 「帮我生成一篇关于XXX的文章」
- 「创建公众号内容」
- 「写一篇公众号文章关于...」
## Language
**Match user's language**: 使用用户使用的语言回复。用户用中文，则用中文回复。
## Preferences (EXTEND.md)
配置文件位于项目 skill 目录下：
```bash
.claude/skills/wechat-article-pipeline/EXTEND.md
.claude/skills/wechat-article-pipeline/.env
```
**首次使用必须配置 EXTEND.md**，包含多账号信息。
### EXTEND.md Schema
```yaml
# ~/.baoyu-skills/wechat-article-pipeline/EXTEND.md
accounts:
  - name: "账号名称"
    app_id: "wx..."           # 可选，用于 API 发布
    app_secret: "..."         # 可选，用于 API 发布
    description: "账号描述"    # 可选，帮助识别
    default_author: "作者名"   # 可选
    default_theme: "default"  # 可选，markdown-to-html 主题
default_account: "账号名称"
default_workflow: "key_checkpoint"  # key_checkpoint | auto | step_by_step
# 图片生成配置（可选）
image_gen:
  provider: "volcengine"      # volcengine | openai | google | dashscope
  api_key: "..."              # 火山引擎 ARK_API_KEY
  model: "doubao-seedream-4-5-251128"  # 模型名称
```
## 整体流程
```
用户输入方向 → [节点①: 选题确定] → 搜索+资料收集 → [节点②: 文章大纲] →
撰写文案 → [节点③: 文章初稿] → 格式化+封面+配图 → [节点④: 格式化预览] →
[节点⑤: 发布前确认] → 保存到公众号草稿箱
```
## Workflow
### Step 0: 启动与配置检查 ⛔ BLOCKING
```
SOP Progress:
- [ ] Step 0: 检查配置 (EXTEND.md) ⛔ BLOCKING
- [ ] Step 1: 选择目标公众号
- [ ] Step 2: 选题头脑风暴 → [节点①]
- [ ] Step 3: 搜索与资料收集
- [ ] Step 4: 生成文章大纲 → [节点②]
- [ ] Step 5: 撰写文章初稿 → [节点③]
- [ ] Step 6: 文本格式化
- [ ] Step 7: 生成封面图
- [ ] Step 8: 生成文章配图 → [节点④]
- [ ] Step 9: 发布前确认 → [节点⑤]
- [ ] Step 10: 保存到公众号草稿箱
```
**检查流程**：
1. 检查 EXTEND.md 是否存在
2. 如不存在，运行首次配置向导
3. 加载账号列表和默认设置
**首次配置向导**（references/config/first-time-setup.md）：
```
欢迎使用微信公众号 SOP！
请配置你的公众号信息：
1. 请输入第一个公众号名称：
2. 请输入 AppID（可选，用于 API 发布）：
3. 请输入默认作者名：
4. 是否添加更多账号？(y/n)
配置将保存到：~/.baoyu-skills/wechat-article-pipeline/EXTEND.md
```
### Step 1: 选择目标公众号
使用 AskUserQuestion 让用户选择目标公众号：
```
请选择要发布到的公众号：
1. [账号A] - 技术分享类
2. [账号B] - 生活随笔类
3. [账号C] - 产品运营类
```
### Step 2: 选题头脑风暴 → [节点①]
**输入**：用户提供模糊方向或话题
**处理**：
1. 分析用户输入的方向
2. 结合目标公众号定位
3. 发散思考，生成 3-5 个具体选题建议
**输出格式**：
```markdown
## 选题建议
### 选题 1: [标题]
- **核心观点**：一句话概括
- **目标读者**：谁会感兴趣
- **内容方向**：主要讲什么
### 选题 2: [标题]
...
### 选题 3: [标题]
...
```
**[节点① 确认]**：使用 AskUserQuestion
```
请选择一个选题：
1. [选题1标题] (推荐)
2. [选题2标题]
3. [选题3标题]
4. 自定义选题 - 我有其他想法
```
### Step 3: 搜索与资料收集
**自动执行（无需确认）**
**处理**：
1. 询问用户是否有参考 URL（可选）
2. 使用 WebSearch 搜索相关话题
3. 如有参考 URL，调用 `baoyu-url-to-markdown` 抓取
4. 整理资料包
**用户交互**：
```
是否有参考资料链接？(可选)
- 输入 URL（多个用空格分隔）
- 或输入 "无" 跳过
```
**输出**：资料包摘要
```markdown
## 资料收集完成
### 网络搜索结果
1. [来源1标题] - 关键信息提取
2. [来源2标题] - 关键信息提取
...
### 参考文章（用户提供的 URL）
1. [文章标题] - 核心观点摘要
...
### 可用素材总结
- 关键数据/统计
- 专家观点
- 案例/故事
```
### Step 4: 生成文章大纲 → [节点②]
**处理**：
1. 基于选题和资料，生成文章结构
2. 包含：标题候选、各章节要点、预计字数
**输出格式**：
```markdown
## 文章大纲
### 标题候选
1. [标题1]
2. [标题2]
3. [标题3]
### 结构
1. **开篇引入** (约200字)
   - 钩子：用什么吸引读者
   - 背景：为什么重要
2. **核心观点一** (约500字)
   - 要点：
   - 案例/数据：
3. **核心观点二** (约500字)
   - 要点：
   - 案例/数据：
4. **核心观点三** (约500字)
   - 要点：
   - 案例/数据：
5. **总结与行动建议** (约200字)
   - 核心回顾
   - 行动呼吁
预计总字数：约 2000 字
```
**[节点② 确认]**：
```
文章大纲已生成，请确认：
1. 确认大纲，开始撰写
2. 修改大纲 - 我有调整建议
3. 重新生成大纲
```
### Step 5: 撰写文章初稿 → [节点③]
**处理**：
根据场景选择写作模式：
- **从零创作**：基于资料包原创
- **改写重组**：用户提供了源文章，进行改写
- **大纲填充**：按大纲结构填充内容
**输出**：完整文章初稿（Markdown 格式）
**[节点③ 确认]**：
```
文章初稿已完成，请审阅：
1. 确认内容，继续格式化
2. 修改内容 - 我有修改意见
3. 重新撰写某些段落
```
**如用户提出修改意见**：
- 记录修改要求
- 修改对应部分
- 再次确认
### Step 6: 文本格式化
**自动执行**
调用 `baoyu-format-markdown` skill：
1. 添加/优化 frontmatter（title, summary, featureImage）
2. 格式化标题层级
3. 添加加粗、列表等格式
4. 中英文间距处理
**输出**：格式化后的 Markdown 文件
### Step 7: 生成封面图
**自动执行**
从 EXTEND.md 读取 `image_gen` 配置，使用火山引擎 API 生成封面图：
**火山引擎 API 调用方式**：
```bash
curl -s "https://ark.cn-beijing.volces.com/api/v3/images/generations" \
  -H "Authorization: Bearer ${ARK_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "doubao-seedream-4-5-251128",
    "prompt": "[图片描述]",
    "size": "2048x2048",
    "response_format": "b64_json"
  }'
```
**输出**：保存到 `imgs/cover.png`
### Step 8: 生成文章配图 → [节点④]
**自动执行**
使用火山引擎 API 生成配图（默认 2-3 张）：
1. 分析文章结构，识别需要配图的位置
2. 为每个位置生成描述性 prompt
3. 调用火山引擎 API 生成图片
4. 保存到 `imgs/01-xxx.png`, `imgs/02-xxx.png` 等
5. 插入到文章中
**[节点④ 确认]**：
```
格式化与配图完成！
封面图：[显示封面缩略图或路径]
配图数量：X 张
请确认视觉效果：
1. 确认，准备发布
2. 重新生成封面图
3. 调整配图
4. 跳过配图，仅保留封面
```
### Step 9: 发布前确认 → [节点⑤]
**整理发布信息**：
```markdown
## 发布信息确认
**目标公众号**：[账号名称]
**文章标题**：[标题]
**文章摘要**：[摘要内容]
**封面图**：imgs/cover.png
**作者**：[作者名]
**字数**：约 X 字
**配图**：X 张
草稿将保存到公众号后台，需手动发布。
```
**[节点⑤ 确认]**：
```
确认发布信息：
1. 确认，保存到草稿箱
2. 修改标题/摘要
3. 更换封面图
4. 取消发布
```
### Step 10: 保存到公众号草稿箱
调用 `baoyu-post-to-wechat` skill：
1. 转换 Markdown 为 HTML
2. 上传封面图获取 media_id
3. 调用微信 API 保存草稿
4. 返回草稿箱链接
**完成报告**：
```
🎉 公众号文章已保存到草稿箱！
**文章信息**
- 标题：[标题]
- 摘要：[摘要]
- 字数：约 X 字
- 配图：封面 + X 张内文图
**目标公众号**：[账号名称]
**下一步**
→ 登录公众号后台确认并发布
→ 草稿箱链接：https://mp.weixin.qq.com
**本地文件**
- 文章：posts/[日期]/[slug]/article.md
- 封面：posts/[日期]/[slug]/imgs/cover.png
- 配图：posts/[日期]/[slug]/imgs/
```
## 依赖 Skills
| Skill | 用途 | 必需 |
|-------|------|------|
| `baoyu-format-markdown` | 文本格式化 | ✅ |
| `baoyu-cover-image` | 封面图生成 | ✅ |
| `baoyu-article-illustrator` | 文章配图 | ✅ |
| `baoyu-post-to-wechat` | 发布到微信 | ✅ |
| `baoyu-url-to-markdown` | URL 内容抓取 | ⭕ 可选 |
**检查依赖**：
```bash
# 检查所有依赖 skills
for skill in baoyu-format-markdown baoyu-cover-image baoyu-article-illustrator baoyu-post-to-wechat; do
  if [ -f "skills/${skill}/SKILL.md" ] || [ -f "$HOME/.claude/skills/${skill}/SKILL.md" ]; then
    echo "✓ ${skill}"
  else
    echo "✗ ${skill} (missing)"
  fi
done
```
如缺少依赖，提示用户安装。
## 输出目录结构
```
posts/
└── YYYY-MM-DD/
    └── {topic-slug}/
        ├── article.md           # 格式化后的文章
        ├── article-formatted.md # 最终版本
        ├── outline.md           # 文章大纲
        ├── research.md          # 资料收集
        └── imgs/
            ├── cover.png        # 封面图
            ├── 01-xxx.png       # 配图1
            ├── 02-xxx.png       # 配图2
            └── ...
```
## 工作流模式
| 模式 | 说明 |
|------|------|
| `key_checkpoint` | 关键节点确认（默认）- 5 个节点暂停 |
| `auto` | 全自动 - 无暂停，直达草稿箱 |
| `step_by_step` | 分步确认 - 每步都暂停 |
通过 EXTEND.md 的 `default_workflow` 配置，或启动时询问。
## 前置条件
### 1. 项目依赖安装
在项目根目录运行：
```bash
# 安装发布和 Markdown 转 HTML 所需的依赖
bun add juice front-matter marked highlight.js reading-time fflate md5 unified remark-parse remark-stringify remark-cjk-friendly
```
或使用 npm：
```bash
npm install juice front-matter marked highlight.js reading-time fflate md5
```
### 2. 微信公众号配置
**IP 白名单**（API 发布必需）：
1. 登录 https://mp.weixin.qq.com
2. 进入「开发」→「基本配置」→「IP白名单」
3. 添加当前公网 IP
**获取当前公网 IP**：
```bash
curl -s ifconfig.me
```
### 3. 图片生成 API（可选）
如需生成封面图和配图，需配置图片生成 API。
**火山引擎（推荐）**：
1. 注册火山引擎账号：https://console.volcengine.com/ark
2. 创建 API Key
3. 开通 `doubao-seedream-4-5-251128` 模型
**配置方式**：
- 在 EXTEND.md 中添加 `image_gen` 配置
- 或首次使用时根据提示配置
## 错误处理
| 情况 | 处理 |
|------|------|
| EXTEND.md 未配置 | 运行首次配置向导 |
| 依赖 skill 缺失 | 提示安装，提供安装指南 |
| **项目依赖缺失** | 提示运行 `bun add juice front-matter ...` |
| 网络搜索失败 | 继续使用用户提供的资料 |
| 图片生成失败 | 重试一次，仍失败则跳过该图 |
| **图片 API 未配置** | 提供配置向导或跳过图片生成 |
| 微信 API 错误 | 检查凭证，提示重新配置 |
| **IP 白名单错误 (40164)** | 提示添加当前 IP 到白名单 |
| **模型未开通** | 提示在控制台开通对应模型 |
## References
| 文档 | 内容 |
|------|------|
| [references/config/first-time-setup.md](references/config/first-time-setup.md) | 首次配置向导 |
| [references/workflow-detail.md](references/workflow-detail.md) | 详细工作流程 |
| [references/writing-modes.md](references/writing-modes.md) | 写作模式说明 |