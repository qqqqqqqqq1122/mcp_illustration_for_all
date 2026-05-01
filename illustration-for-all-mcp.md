# MCP（Model Context Protocol）服务器工具完整说明书

> **数据来源：** 官方 GitHub 仓库 [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)、[modelcontextprotocol/servers-archived](https://github.com/modelcontextprotocol/servers-archived)、[mcpservers.org](https://mcpservers.org)、[glama.ai/mcp/servers](https://glama.ai/mcp/servers)  
> **生成日期：** 2026-05-01  
> **来源最后更新时间：** 2025 年（仓库持续维护中）

---

## 说明

- **服务器类型**：`本地运行` = 服务运行在用户本机，无需外部账号；`需远程服务` = 需要外部 API Key 或云服务账号。
- **发布日期**：MCP 协议由 Anthropic 于 **2024 年 11 月 25 日**正式发布，所有官方参考实现服务器均于同日首发，后续持续迭代。
- 表中"—"表示该字段在官方文档中无对应限制说明或不适用。

---

## 目录

1. [Fetch（网页内容抓取）](#1-fetch-服务器)
2. [Filesystem（本地文件系统）](#2-filesystem-服务器)
3. [Git（Git 仓库操作）](#3-git-服务器)
4. [Memory（知识图谱记忆）](#4-memory-服务器)
5. [Sequential Thinking（顺序思维）](#5-sequential-thinking-服务器)
6. [Time（时间与时区）](#6-time-服务器)
7. [AWS KB Retrieval（AWS 知识库检索）](#7-aws-kb-retrieval-服务器)
8. [Brave Search（Brave 搜索）](#8-brave-search-服务器)
9. [GitHub（GitHub API）](#9-github-服务器)
10. [GitLab（GitLab API）](#10-gitlab-服务器)
11. [Google Drive（谷歌云盘）](#11-google-drive-服务器)
12. [Google Maps（谷歌地图）](#12-google-maps-服务器)
13. [PostgreSQL（PostgreSQL 数据库）](#13-postgresql-服务器)
14. [Puppeteer（浏览器自动化）](#14-puppeteer-服务器)
15. [Redis（Redis 数据库）](#15-redis-服务器)
16. [Sentry（错误监控）](#16-sentry-服务器)
17. [Slack（即时通讯）](#17-slack-服务器)
18. [SQLite（SQLite 数据库）](#18-sqlite-服务器)

---

## 1. Fetch 服务器

- **网址：** https://github.com/modelcontextprotocol/servers/tree/main/src/fetch  
- **服务器类型：** 本地运行  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `fetch` | 从互联网获取指定 URL 的页面内容，默认自动将 HTML 转换为 Markdown 格式，方便 LLM 阅读；支持分块读取（通过 `start_index` 参数）；可设定最大返回字符数（`max_length`，默认 5000）；支持返回原始内容（`raw=true`）。 | ① 无法突破网站访问控制、登录墙或验证码；② 默认遵守目标站点的 `robots.txt` 限制（可通过 `--ignore-robots-txt` 关闭）；③ 有访问本地/内网 IP 地址的安全风险，需谨慎配置；④ 不支持 JavaScript 动态渲染页面（建议改用 Puppeteer）；⑤ 单次最多返回约 5000 字符（可配置）。 | 让 LLM 实时阅读网页文档、博客文章、API 说明；抓取公开数据进行分析；辅助 Agent 完成"查看某页面内容"任务。 |

---

## 2. Filesystem 服务器

- **网址：** https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem  
- **服务器类型：** 本地运行  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `read_text_file` | 读取文件的完整文本内容（UTF-8）；支持 `head`（前 N 行）和 `tail`（后 N 行）参数。 | 仅限在已授权目录内操作；无法同时指定 head 和 tail；不支持二进制文件直接读取。 | 查阅代码文件、日志、配置文件内容。 |
| `read_media_file` | 读取图片或音频文件，以 base64 编码 + MIME 类型的方式返回。 | 仅限已授权目录；大型媒体文件可能导致传输数据量过大。 | 让 LLM 查看本地图片、分析图表。 |
| `read_multiple_files` | 同时读取多个文件；单个文件读取失败不会中断整个操作。 | 受目录授权限制；所有文件必须在允许目录范围内。 | 批量检查多个配置文件或源码文件。 |
| `write_file` | 新建文件或覆盖已有文件，写入指定内容。 | **破坏性操作**：会覆盖已有文件；受目录授权限制；无法追加写入（须先读取再合并后写入）。 | 生成代码文件、保存分析结果、创建配置文件。 |
| `edit_file` | 对文件进行精确的行级或多行文本替换编辑；支持 `dryRun` 预览模式；保留缩进风格；返回 git-style diff 对比结果。 | 破坏性操作（修改不可逆）；匹配失败则不执行；建议先用 `dryRun=true` 预览。 | 自动修复代码 bug、批量替换文本、更新配置项。 |
| `create_directory` | 创建目录（含父目录），目录已存在时静默成功。 | 受目录授权限制；无法在授权范围外创建。 | 初始化项目目录结构。 |
| `list_directory` | 列出目录内容，标注 `[FILE]` / `[DIR]` 前缀。 | 受目录授权限制；不显示隐藏文件（视操作系统而定）。 | 浏览项目文件结构。 |
| `list_directory_with_sizes` | 同 `list_directory`，但附带文件大小信息，支持按名称或大小排序，并显示统计汇总。 | 受目录授权限制。 | 分析目录占用空间，排查大文件。 |
| `move_file` | 移动或重命名文件/目录；目标路径已存在时报错。 | 破坏性操作（删除源文件）；目标已存在则失败；受目录授权限制。 | 文件整理、重构项目目录。 |
| `search_files` | 在指定目录下递归搜索符合 glob 模式的文件/目录，支持排除模式。 | 受目录授权限制；glob 模式匹配，不支持内容全文搜索（内容搜索须配合 `read_text_file`）。 | 在大型项目中查找特定文件。 |
| `directory_tree` | 递归返回目录树的 JSON 结构，包含名称和类型，支持排除模式。 | 受目录授权限制；大型目录返回数据量较大。 | 全局浏览项目结构，生成目录文档。 |
| `get_file_info` | 获取文件/目录元数据：大小、创建时间、修改时间、访问时间、类型、权限。 | 受目录授权限制；仅返回元数据，不读取内容。 | 检查文件是否最新、审查权限设置。 |
| `list_allowed_directories` | 列出当前服务器被授权访问的所有目录。 | 只读，无参数。 | 确认服务器访问范围。 |

---

## 3. Git 服务器

- **网址：** https://github.com/modelcontextprotocol/servers/tree/main/src/git  
- **服务器类型：** 本地运行  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `git_status` | 显示 Git 仓库工作目录的当前状态（修改、暂存、未跟踪文件）。 | 仅限本地仓库；不连接远程仓库。 | 在提交前检查改动。 |
| `git_diff_unstaged` | 显示工作目录中尚未暂存的更改差异；支持 `context_lines` 参数（默认 3 行上下文）。 | 仅显示未暂存变更；不含已暂存内容。 | 代码审查，了解当前编辑内容。 |
| `git_diff_staged` | 显示已暂存（将被提交）的文件差异。 | 仅显示已暂存变更。 | 提交前最终检查。 |
| `git_diff` | 比较当前分支与指定分支或 commit 之间的差异。 | 需指定 `target` 参数（分支名或 commit hash）。 | 对比分支差异、Review 变更范围。 |
| `git_commit` | 将已暂存的变更提交到本地仓库；返回新的 commit hash。 | 仅提交已暂存文件；不推送到远程；需先用 `git_add` 暂存文件。 | 自动化 commit 工作流。 |
| `git_add` | 将指定文件添加到暂存区（stage）。 | 仅接受文件路径数组；不支持 glob 模式（须明确列出文件）。 | 为提交准备文件。 |
| `git_reset` | 清空暂存区（unstage 所有已暂存文件），工作目录文件不受影响。 | 不回滚工作目录的实际修改。 | 取消误暂存的文件。 |
| `git_log` | 查看提交历史；支持数量限制（`max_count`）和时间范围过滤（ISO 8601 / 相对时间）。 | 仅本地历史；显示字段为 hash、作者、日期、消息。 | 追溯变更历史、生成 changelog。 |
| `git_create_branch` | 基于当前或指定分支创建新分支。 | 仅本地操作；不推送远程；分支名不能与已有分支重复。 | 为新功能或 bugfix 创建分支。 |
| `git_checkout` | 切换到指定分支。 | 工作目录有未提交更改时可能失败；仅本地分支。 | 在分支间切换工作。 |
| `git_show` | 显示指定 commit / branch / tag 的详细内容（变更 diff + 元数据）。 | 需指定有效的 revision。 | 审查特定 commit 的改动详情。 |
| `git_branch` | 列出分支（本地 / 远程 / 全部）；支持按 commit SHA 过滤包含或不包含该 commit 的分支。 | 远程分支需 fetch 后才能看到；不执行 fetch 操作。 | 管理分支，了解项目分支全貌。 |

---

## 4. Memory 服务器

- **网址：** https://github.com/modelcontextprotocol/servers/tree/main/src/memory  
- **服务器类型：** 本地运行  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `create_entities` | 在知识图谱中创建多个实体，每个实体含名称、类型（如 person/organization）和观察列表。已存在同名实体则忽略。 | 实体名称必须唯一；不支持实体更新（只能新增或删除后重建）。 | 记录用户偏好信息、项目成员信息。 |
| `create_relations` | 在实体间创建有向关系（主动语态），关系类型自定义（如 works_at、depends_on）。 | 重复关系将被跳过；关系两端的实体须已存在。 | 构建人际关系图、软件依赖图谱。 |
| `add_observations` | 向已有实体追加新的观察（事实）条目，每条观察为一个独立字符串。 | 目标实体必须已存在；否则报错。 | 持续积累关于某人/项目的新信息。 |
| `delete_entities` | 删除指定实体及其所有关联关系（级联删除）；实体不存在时静默成功。 | 操作不可逆；会同时删除所有关联关系。 | 清理过期信息。 |
| `delete_observations` | 删除指定实体上的特定观察条目；不存在时静默成功。 | 只删除观察，不影响实体本身或关系。 | 更正错误的历史观察记录。 |
| `delete_relations` | 删除特定关系（from / to / relationType 三元组精确匹配）；不存在时静默成功。 | 需精确匹配三个字段。 | 移除已失效的关系。 |
| `read_graph` | 读取整个知识图谱，返回所有实体和关系的完整结构。 | 数据量大时返回内容较多；无过滤参数。 | 全局浏览记忆内容，做出上下文推断。 |
| `search_nodes` | 按查询字符串搜索节点，匹配范围包括实体名称、实体类型、观察内容。 | 全文模糊搜索；不支持正则或高级查询语法。 | 快速定位某个人或项目的相关信息。 |
| `open_nodes` | 按名称列表精确获取多个节点及其相互关系。 | 不存在的节点被静默跳过；返回仅限请求节点间的关系。 | 获取指定实体的详细信息用于上下文注入。 |

---

## 5. Sequential Thinking 服务器

- **网址：** https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking  
- **服务器类型：** 本地运行  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `sequential_thinking` | 为 LLM 提供结构化的逐步思考框架：记录每一步思考内容（`thought`）、当前思考编号（`thoughtNumber`）、预估总思考数（`totalThoughts`）；支持修正之前的思考（`isRevision`）；支持分支推理（`branchFromThought`）；当需要时可动态扩展总思考数（`needsMoreThoughts`）。 | ① 工具本身不执行任何外部动作，只辅助 LLM 组织思路；② 总思考数是估算值，可能需要多次调整；③ 不提供外部数据获取能力，需配合其他工具使用。 | 复杂问题分步推导（如系统架构设计、数学证明）；需要反复修正假设的分析任务；多方案对比决策；调试生产问题的根因分析。 |

---

## 6. Time 服务器

- **网址：** https://github.com/modelcontextprotocol/servers/tree/main/src/time  
- **服务器类型：** 本地运行  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `get_current_time` | 获取指定 IANA 时区（如 `America/New_York`、`Asia/Shanghai`）的当前时间，返回包含 ISO 8601 格式时间和是否夏令时（`is_dst`）的 JSON 对象；可自动检测系统时区。 | 精度为秒级；不支持历史时间查询；IANA 时区名须正确拼写。 | 询问"北京现在几点"；在跨时区 Agent 工作流中注入准确时间戳。 |
| `convert_time` | 将一个时区的 HH:MM 格式时间转换为另一个时区的时间，返回源时区和目标时区的完整时间信息及时差（`time_difference`）。 | 仅支持 24 小时制输入（HH:MM）；不处理日期，只处理时刻；需提供有效 IANA 时区名。 | 安排国际会议（"纽约下午 4 点对应东京几点"）；计算跨时区工作时间。 |

---

## 7. AWS KB Retrieval 服务器

> ⚠️ **已归档**（Archived）。迁移至 [servers-archived](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/aws-kb-retrieval-server)，建议评估替代方案。

- **网址：** https://github.com/modelcontextprotocol/servers-archived/tree/main/src/aws-kb-retrieval-server  
- **服务器类型：** 需远程服务（需 AWS 账号及 Bedrock Agent Runtime 权限）  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `retrieve_from_aws_kb` | 通过 AWS Bedrock Agent Runtime 对指定 AWS Knowledge Base 执行 RAG（检索增强生成）检索，返回最相关的 N 条文档片段（默认 3 条）。参数：`query`（查询字符串）、`knowledgeBaseId`（Knowledge Base ID）、`n`（结果数，默认 3）。 | ① 需要有效的 AWS 凭据（Access Key + Secret Key + Region）；② 需在 AWS Bedrock 中已创建 Knowledge Base；③ 受 AWS 配额和费用限制；④ 仅检索，不修改知识库内容；⑤ 不支持跨 Knowledge Base 联合查询。 | 企业内部文档问答；基于 AWS S3/RDS 中的私有数据做智能检索；将 AWS 知识库与 Claude 集成以提供专业领域回答。 |

---

## 8. Brave Search 服务器

> ⚠️ **已归档**。官方维护已迁至 [brave/brave-search-mcp-server](https://github.com/brave/brave-search-mcp-server)。

- **网址：** https://github.com/brave/brave-search-mcp-server  
- **服务器类型：** 需远程服务（需 Brave Search API Key）  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `brave_web_search` | 执行 Brave 网页搜索，支持分页（`offset`，最多 9 页）和结果数量控制（`count`，最多 20 条/页）。返回网页搜索结果列表（含标题、摘要、URL）。 | ① 需要 Brave Search API Key（免费版每月 2000 次查询）；② 每页最多 20 条结果；③ 最多翻 9 页（offset ≤ 9）；④ 不返回完整网页内容（只返回摘要）。 | 让 LLM 实时联网查询最新信息；新闻搜索；研究辅助。 |
| `brave_local_search` | 搜索本地商家和服务（餐厅、店铺等），附带地址、评分、营业时间等详细信息；无本地结果时自动降级为网页搜索。 | 本地搜索效果依赖查询中包含地理位置信息；需 API Key；最多 20 条结果。 | 查找附近餐厅/服务；旅行规划；本地生活服务推荐。 |

---

## 9. GitHub 服务器

> ⚠️ **已归档**。官方维护已迁至 [github/github-mcp-server](https://github.com/github/github-mcp-server)。

- **网址：** https://github.com/github/github-mcp-server  
- **服务器类型：** 需远程服务（需 GitHub Personal Access Token）  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `create_or_update_file` | 在指定仓库和分支创建或更新单个文件；不存在的分支会自动创建；需提供 commit message。 | 需要仓库写权限；更新现有文件时建议提供 `sha` 避免冲突；不支持二进制文件。 | 自动提交代码变更；文档自动更新。 |
| `push_files` | 批量推送多个文件为单次 commit，包含文件路径和内容数组。 | 需要仓库写权限；所有文件必须在同一 commit 中提交。 | 一次性推送多文件变更；初始化仓库文件。 |
| `search_repositories` | 按关键词搜索 GitHub 仓库，支持分页（`page`/`perPage`，最多 100/页）。 | 受 GitHub API 速率限制；搜索结果为公开仓库，私有仓库搜索需相应权限。 | 查找开源项目；技术选型调研。 |
| `create_repository` | 创建新的 GitHub 仓库，支持设置描述、公私有、是否初始化 README。 | 需有创建仓库权限；仓库名在同一 owner 下唯一。 | 自动化项目初始化。 |
| `get_file_contents` | 获取指定仓库中文件或目录的内容，支持指定分支。 | 大型文件可能被截断（GitHub API 限制 1MB）；需要相应读取权限。 | 读取远程配置文件；代码审查。 |
| `create_issue` | 在仓库中创建 Issue，支持标题、描述、分配人、标签、里程碑。 | 需要仓库 Issue 写权限；标签和里程碑须已存在。 | 自动创建 bug 报告；任务跟踪。 |
| `create_pull_request` | 创建 PR，支持标题、描述、源分支、目标分支、是否为草稿 PR。 | 需要仓库写权限；源分支和目标分支须已存在且有差异。 | 自动化 PR 创建；代码审查流程集成。 |
| `fork_repository` | Fork 指定仓库到当前用户或指定组织。 | Fork 目标不能已存在同名仓库；受 GitHub API 速率限制。 | 贡献开源项目前 fork 仓库。 |
| `create_branch` | 在仓库中创建新分支，可选择基础分支（默认为仓库默认分支）。 | 分支名须符合 Git 规范；不能与已有分支重名。 | 为新功能自动创建分支。 |
| `list_issues` | 列出仓库 Issue，支持状态过滤（open/closed/all）、标签过滤、排序、时间过滤。 | 受分页限制；每页最多 100 条。 | 项目管理看板；统计 Issue 数量。 |
| `update_issue` | 更新已有 Issue 的标题、描述、状态、标签、分配人、里程碑。 | 需要 Issue 写权限；Issue 编号须存在。 | 自动关闭已解决的 Issue；批量更新标签。 |
| `add_issue_comment` | 向指定 Issue 添加评论。 | 需要评论写权限；Issue 须存在且开放评论。 | 自动报告测试结果；机器人回复。 |
| `search_code` | 在 GitHub 全平台搜索代码（支持 GitHub 代码搜索语法）；支持分页，最多 100/页。 | 受 GitHub 代码搜索 API 速率限制；只返回匹配行，不返回完整文件。 | 跨仓库查找函数实现；安全漏洞模式扫描。 |
| `search_issues` | 搜索 Issues 和 PR（支持 GitHub issues 搜索语法）；支持多种排序方式。 | 受 GitHub API 速率限制；最多 100/页。 | 查找相关 bug 讨论；追踪特定功能请求。 |
| `search_users` | 搜索 GitHub 用户，支持 followers/repositories/joined 等排序。 | 受 GitHub API 速率限制；私人账号信息有限。 | 查找贡献者；人才技术背景调研。 |
| `list_commits` | 列出仓库指定分支的提交历史，支持分页。 | 需要仓库读取权限；每页最多 100 条。 | 追踪代码变更；生成 changelog。 |
| `get_issue` | 获取指定 Issue 的详细信息（标题、描述、状态、标签等）。 | 需要对应仓库的读取权限。 | 自动获取 Issue 详情用于分析或 AI 回复。 |
| `get_pull_request` | 获取指定 PR 的详细信息（含 diff 和 review 状态）。 | 需要仓库读取权限；大型 PR 的 diff 可能被截断。 | 自动化代码 Review；PR 状态监控。 |
| `list_pull_requests` | 列出仓库 PR，支持状态/分支/排序过滤，最多 100/页。 | 需要仓库读取权限。 | 项目 PR 管理；统计 PR 合并率。 |
| `create_pull_request_review` | 对 PR 提交 Review（APPROVE/REQUEST_CHANGES/COMMENT），支持行级 review 注释。 | 需要 PR review 权限；不能 approve 自己的 PR。 | 自动化代码审查；LLM 辅助 review。 |
| `merge_pull_request` | 合并指定 PR（支持 merge/squash/rebase 三种合并方式）。 | 需要仓库写权限；PR 须通过所有 CI 检查和保护规则。 | 自动化发布流程；条件满足后自动合并。 |

---

## 10. GitLab 服务器

> ⚠️ **已归档**。迁至 [servers-archived](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/gitlab)。

- **网址：** https://github.com/modelcontextprotocol/servers-archived/tree/main/src/gitlab  
- **服务器类型：** 需远程服务（需 GitLab Personal Access Token）  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `create_or_update_file` | 在 GitLab 项目中创建或更新单个文件，支持重命名（`previous_path`），不存在的分支自动创建。 | 需要项目写权限；需提供 commit message。 | 自动提交代码或文档变更至 GitLab。 |
| `push_files` | 批量推送多文件为单次 commit。 | 需要项目写权限；所有文件在同一 commit 中。 | 批量文件更新；项目初始化。 |
| `search_repositories` | 搜索 GitLab 项目，支持分页（默认 20/页）。 | 搜索范围取决于 Token 权限（公开/私有项目）。 | 查找组织内已有项目；技术调研。 |
| `create_repository` | 创建新的 GitLab 项目，支持可见性（private/internal/public）和初始化 README。 | 需有创建项目权限；同命名空间下名称唯一。 | 新项目脚手架；批量创建项目。 |
| `get_file_contents` | 获取项目文件或目录内容，支持指定分支/tag/commit。 | 受 GitLab API 文件大小限制；需读取权限。 | 远程读取配置文件；代码分析。 |
| `create_issue` | 在项目中创建 Issue，支持标题、描述、分配人、标签、里程碑。 | 需要 Issue 写权限；标签和里程碑须已存在。 | Bug 报告自动化；任务管理。 |
| `create_merge_request` | 创建 Merge Request，支持标题、描述、源/目标分支、草稿状态、允许协作提交。 | 需要项目写权限；源/目标分支须已存在。 | 自动化 MR 创建；CI/CD 集成。 |
| `fork_repository` | Fork 项目到指定命名空间。 | 目标命名空间下不能已有同名项目。 | 开源贡献工作流。 |
| `create_branch` | 创建新分支，支持指定源分支/commit/tag。 | 分支名须唯一且符合 Git 规范。 | 功能开发分支创建。 |

---

## 11. Google Drive 服务器

> ⚠️ **已归档**。迁至 [servers-archived](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/gdrive)。

- **网址：** https://github.com/modelcontextprotocol/servers-archived/tree/main/src/gdrive  
- **服务器类型：** 需远程服务（需 Google OAuth 2.0 授权）  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `search` | 在 Google Drive 中按关键词搜索文件，返回文件名和 MIME 类型。支持通过资源 URI（`gdrive:///<file_id>`）读取文件内容；Google Docs 自动导出为 Markdown、Sheets 导出为 CSV、Presentations 导出为纯文本、Drawings 导出为 PNG。 | ① 仅有只读权限（`drive.readonly`）；② 不支持上传、编辑、删除文件；③ 首次使用须完成 OAuth 2.0 授权流程；④ 搜索结果有数量限制；⑤ 不能搜索其他用户私有文件。 | AI 助理读取 Google Docs 文档并总结；在 Drive 中检索项目资料；将 Sheets 数据导出分析。 |

---

## 12. Google Maps 服务器

> ⚠️ **已归档**。迁至 [servers-archived](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/google-maps)。

- **网址：** https://github.com/modelcontextprotocol/servers-archived/tree/main/src/google-maps  
- **服务器类型：** 需远程服务（需 Google Maps API Key）  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `maps_geocode` | 将地址文本转换为 GPS 坐标（正向地理编码），返回经纬度、格式化地址和 `place_id`。 | 需要 Google Maps API Key；免费配额有限；地址描述越精确结果越准确。 | 获取地址坐标用于路线规划或可视化。 |
| `maps_reverse_geocode` | 将经纬度坐标转换为地址（反向地理编码），返回格式化地址、`place_id` 和地址组成信息。 | 需要 API Key；结果精度取决于 Google Maps 数据覆盖情况。 | 将 GPS 轨迹点转换为可读地址。 |
| `maps_search_places` | 通过文本查询搜索地点，支持指定中心位置和半径（最大 50000 米）。返回地点名称、地址、坐标。 | 需要 API Key；半径上限 50000 米；搜索结果数量有限。 | 搜索附近餐厅、景点；旅行规划。 |
| `maps_place_details` | 获取指定 `place_id` 的详细信息（名称、地址、联系方式、评分、评论、营业时间）。 | 需要 API Key；`place_id` 须有效；详细信息取决于商家录入情况。 | 获取商家详情；智能助手回答"这家餐厅几点关门"。 |
| `maps_distance_matrix` | 计算多个出发地到多个目的地的距离和时间矩阵，支持驾车/步行/骑行/公交四种交通方式。 | 需要 API Key；origins 和 destinations 数量乘积不宜过大（API 限制）；实时路况受数据更新延迟影响。 | 物流配送路径优化；多地点会面地点选择。 |
| `maps_elevation` | 获取指定地理坐标数组的海拔高度数据。 | 需要 API Key；精度取决于 Google 地形数据分辨率。 | 路线海拔分析；地形可视化。 |
| `maps_directions` | 获取两点间的详细导航路线（含分步说明、总距离、预计时间），支持四种交通方式。 | 需要 API Key；公交路线依赖当地 GTFS 数据；实时路况有一定延迟。 | 智能助手提供出行建议；自动规划旅行路线。 |

---

## 13. PostgreSQL 服务器

> ⚠️ **已归档**。迁至 [servers-archived](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/postgres)。

- **网址：** https://github.com/modelcontextprotocol/servers-archived/tree/main/src/postgres  
- **服务器类型：** 本地运行（需本地或可访问的 PostgreSQL 实例）  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `query` | 在 PostgreSQL 数据库中执行 SELECT 查询（只读事务），返回查询结果的 JSON 数组；同时通过资源接口（`postgres://<host>/<table>/schema`）暴露表的结构信息（列名和数据类型）。 | ① **只读**：所有查询在 READ ONLY 事务中执行，无法执行 INSERT/UPDATE/DELETE/DDL；② 查询结果大小受 MCP 消息限制；③ 需要数据库连接 URL 和对应权限；④ 不支持存储过程调用。 | 让 LLM 分析业务数据；自然语言转 SQL 查询；报表生成；数据库结构探索。 |

---

## 14. Puppeteer 服务器

> ⚠️ **已归档**。迁至 [servers-archived](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/puppeteer)。

- **网址：** https://github.com/modelcontextprotocol/servers-archived/tree/main/src/puppeteer  
- **服务器类型：** 本地运行（在本机启动真实浏览器）  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `puppeteer_navigate` | 在浏览器中导航到指定 URL；支持自定义 Puppeteer 启动选项（`launchOptions`）。 | ⚠️ 安全警告：可访问本地文件和内网 IP；更改 launchOptions 会重启浏览器；`allowDangerous=false` 时禁止高危启动参数。 | 打开网页；登录后抓取需认证的页面。 |
| `puppeteer_screenshot` | 对当前页面或指定 CSS 选择器元素截图，可自定义宽高；支持 base64 编码输出。 | 截图分辨率有限；元素须在页面中可见（非 display:none）。 | 生成网页快照；视觉回归测试；让 LLM 观察页面状态。 |
| `puppeteer_click` | 点击页面上指定 CSS 选择器的元素。 | 元素须存在且可交互；不支持右键点击（用 button 参数扩展）。 | 自动填表、点击按钮、触发交互。 |
| `puppeteer_hover` | 将鼠标悬停在指定 CSS 选择器的元素上（触发 hover 效果）。 | 元素须存在。 | 触发下拉菜单；显示 tooltip 后截图。 |
| `puppeteer_fill` | 在输入框中填入指定内容（清空后填写）。 | 元素须为输入控件（input/textarea）；不触发复杂输入事件。 | 自动填写表单、搜索框。 |
| `puppeteer_select` | 在 `<select>` 下拉框中选择指定选项值。 | 元素须为 SELECT 标签；value 须与选项值精确匹配。 | 自动化表单中的下拉选择。 |
| `puppeteer_evaluate` | 在浏览器上下文中执行任意 JavaScript 代码并返回结果；同时通过资源接口暴露控制台日志和截图。 | ⚠️ 高风险：可执行任意 JS；须确保输入可信；执行结果受沙盒限制（如浏览器跨域策略）。 | 抓取动态渲染数据；操作 DOM；提取页面变量值。 |

---

## 15. Redis 服务器

> ⚠️ **已归档**。迁至 [servers-archived](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/redis)。

- **网址：** https://github.com/modelcontextprotocol/servers-archived/tree/main/src/redis  
- **服务器类型：** 本地运行（需本地或可访问的 Redis 实例）  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `set` | 在 Redis 中设置键值对；支持可选的过期时间（`expireSeconds`）。 | 值类型仅为字符串；不支持 Hash/List/Set 等复杂数据类型；需本地 Redis 运行中。 | 缓存临时数据；存储会话状态；设置带过期的令牌。 |
| `get` | 按键获取 Redis 中存储的值。 | 键不存在时返回 null；仅支持字符串值类型。 | 读取缓存数据；检查键是否存在。 |
| `delete` | 删除一个或多个 Redis 键（支持字符串或字符串数组）。 | 操作不可逆；批量删除时部分键不存在不报错。 | 清除缓存；删除过期令牌。 |
| `list` | 列出符合 glob 模式的 Redis 键（默认 `*` 列出所有键）。 | 生产环境中 `LIST *` 会阻塞 Redis，不建议在大型数据库上使用；仅返回键名，不返回值。 | 调试键名分布；检查特定前缀的键。 |

---

## 16. Sentry 服务器

> ⚠️ **已归档**。迁至 [servers-archived](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/sentry)。

- **网址：** https://github.com/modelcontextprotocol/servers-archived/tree/main/src/sentry  
- **服务器类型：** 需远程服务（需 Sentry.io Auth Token）  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `get_sentry_issue` | 根据 Sentry Issue ID 或 URL 检索并分析 Sentry 错误报告，返回标题、ID、状态、严重等级、首次/最后发现时间、事件数量和完整调用栈（stacktrace）。 | ① 需要有效的 Sentry Auth Token 和对应项目的访问权限；② 只读，不能修改或关闭 Issue；③ 不能列出 Issue 列表（只能按 ID/URL 查询单个）；④ 不支持跨组织查询。 | AI 辅助 Bug 分析：自动读取错误堆栈并给出修复建议；将 Sentry 告警与 Claude 集成实现智能 oncall 响应。 |

---

## 17. Slack 服务器

> ⚠️ **已归档**。现由 [Zencoder](https://github.com/zencoderai/slack-mcp-server) 维护。

- **网址：** https://github.com/zencoderai/slack-mcp-server  
- **服务器类型：** 需远程服务（需 Slack Bot Token 和 Team ID）  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `slack_list_channels` | 列出工作区中的公开频道（或预配置频道），支持分页（最多 200/页）。返回频道 ID 和基本信息。 | 只能列出 Bot 有权访问的频道；私有频道需被邀请；不返回私信（DM）。 | 了解工作区频道结构；选择要发送消息的频道。 |
| `slack_post_message` | 向指定频道（`channel_id`）发送新消息。 | 需要 `chat:write` 权限；Bot 须已加入该频道；不支持发送文件附件（仅文本）。 | AI 助理主动发送通知；自动化报告推送。 |
| `slack_reply_to_thread` | 在指定消息的线程（thread）中回复。 | 需要 `chat:write` 权限和有效的 `thread_ts`（消息时间戳）；只能回复，不能编辑已有消息。 | 跟进讨论；在 CI/CD 通知线程中追加结果。 |
| `slack_add_reaction` | 为指定消息添加 emoji 表情回应（reaction）。 | 需要 `reactions:write` 权限；emoji 名称须有效（不含冒号）；每条消息对同一 emoji 只能 react 一次。 | 自动对重要通知标记 ✅ 或 👀；情绪化反馈自动化。 |
| `slack_get_channel_history` | 获取指定频道的最近消息（默认 10 条，可配置）。 | 需要 `channels:history` 权限；只能获取 Bot 有权访问的频道；不含线程内回复。 | 监控频道动态；分析近期讨论内容。 |
| `slack_get_thread_replies` | 获取指定消息线程的所有回复列表。 | 需要 `channels:history` 权限；需提供父消息的 `thread_ts`。 | 读取完整讨论线程；汇总会议记录。 |
| `slack_get_users` | 获取工作区用户列表（含基本个人资料），支持分页（最多 200/页）。 | 需要 `users:read` 权限；不含已停用账号的详细信息；不含私信记录。 | 查找工作区成员；@提及自动补全。 |
| `slack_get_user_profile` | 获取指定用户（`user_id`）的详细个人资料（姓名、邮箱、职位等）。 | 需要 `users.profile:read` 权限；部分字段可能为空（用户未填写）。 | 了解团队成员信息；个性化消息推送。 |

---

## 18. SQLite 服务器

> ⚠️ **已归档**。迁至 [servers-archived](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/sqlite)。

- **网址：** https://github.com/modelcontextprotocol/servers-archived/tree/main/src/sqlite  
- **服务器类型：** 本地运行（操作本机 SQLite 文件）  
- **首次发布日期：** 2024-11-25

| 工具名称 | 功能描述（能干什么） | 能力边界（不能干什么 / 限制） | 典型应用场景 |
|----------|----------------------|-------------------------------|--------------|
| `read_query` | 执行 SELECT SQL 查询，返回结果的 JSON 数组。 | 只能执行 SELECT，不能修改数据；查询须为有效 SQL；大结果集可能受内存限制。 | 数据查询分析；报表生成；自然语言转 SQL。 |
| `write_query` | 执行 INSERT、UPDATE 或 DELETE 语句，返回受影响行数（`affected_rows`）。 | 不能执行 DDL（CREATE/DROP）；操作不可撤销（SQLite 默认无事务隔离）。 | 数据录入；状态更新；数据清理。 |
| `create_table` | 执行 CREATE TABLE 语句创建新表。 | 须为有效的 CREATE TABLE SQL；表名不能重复（除非使用 IF NOT EXISTS）。 | 初始化数据库结构；动态创建分析临时表。 |
| `list_tables` | 列出数据库中所有表名，无需参数。 | 只返回表名数组，不含视图或临时表。 | 了解数据库结构；引导 LLM 选择正确的表。 |
| `describe_table` | 获取指定表的 Schema 信息（列名和数据类型数组）。 | 只返回列结构，不返回索引、约束等信息。 | 辅助 LLM 构建正确的 SQL 查询。 |
| `append_insight` | 将业务洞察字符串追加到内置的 memo 资源（`memo://insights`）中，并触发资源更新通知。 | 仅追加，不能删除或修改已有洞察；洞察内容仅为字符串，无结构化格式。 | 在数据分析过程中实时积累发现；生成分析报告摘要。 |

---

## 附录：MCP 生态核心资源导航

| 资源名称 | 网址 | 说明 |
|----------|------|------|
| MCP 官方注册表 | https://registry.modelcontextprotocol.io | Anthropic 官方 MCP Server 注册中心（最新） |
| modelcontextprotocol/servers | https://github.com/modelcontextprotocol/servers | 官方参考实现仓库（活跃维护） |
| modelcontextprotocol/servers-archived | https://github.com/modelcontextprotocol/servers-archived | 官方已归档参考实现 |
| github/github-mcp-server | https://github.com/github/github-mcp-server | GitHub 官方维护的 GitHub MCP Server |
| Awesome MCP Servers (wong2) | https://mcpservers.org | 社区最大聚合目录，按类别分类 |
| Glama.ai MCP Marketplace | https://glama.ai/mcp/servers | 可视化 MCP 市场，支持预览工具函数 |
| Smithery.ai | https://smithery.ai | MCP "应用商店"，支持一键安装配置 |
| TensorBlock/awesome-mcp-servers | https://github.com/TensorBlock/awesome-mcp-servers | GitHub 上最全面的社区汇总列表 |
| punkpeye/awesome-mcp-servers | https://github.com/punkpeye/awesome-mcp-servers | 另一个广泛引用的社区 awesome 列表 |
| mcp-get | https://mcp-get.com | MCP Server 命令行安装管理工具 |

---

*本文档基于 2026-05-01 公开数据编制。MCP 生态快速发展，建议定期查阅上方资源获取最新信息。*
