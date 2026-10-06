# 技术栈

## 基础选择

采用 TypeScript 主栈和 SQLite 主数据库。基线优先满足个人桌面使用、独立 Agent 运行以及外部 Agent 接入。

以下确定技术方向，具体依赖版本在工程初始化时验证并锁定。目前没有已安装依赖或兼容性验收结果。

| 部分 | 选择 | 职责 |
| --- | --- | --- |
| 语言与运行时 | TypeScript strict、Node.js LTS | 核心业务、Agent 运行、记忆与接入代码。 |
| 包管理与工程组织 | pnpm workspace | 管理需要共同开发的应用与模块；具体目录在工程 Plan 中确定。 |
| 桌面端 | Electron | 桌面窗口及本地系统接入。 |
| 界面与构建 | React、electron-vite | 对话、资料、记忆及任务界面，桌面工程构建。 |
| 节点编辑 | React Flow | 工作流节点、连线和画布交互。 |
| 模型调用 | AI SDK Core | 模型适配、流式输出和工具调用基础能力。 |
| Agent 运行 | 项目自身的薄运行核心 | 组织上下文、工具、Skill、任务状态和循环控制。 |
| HTTP 服务 | Hono | RainbowFish 的 HTTP 接入。 |
| 外部 Agent 工具接入 | MCP TypeScript SDK | 提供记忆与会话相关工具。 |
| 宿主适配 | 对应客户端的 Hooks 或扩展 | 处理实际支持的采集与上下文补充时机。 |
| 数据访问与迁移 | better-sqlite3、Drizzle ORM / Kit | SQLite 访问、结构化查询和数据迁移。 |
| 主数据库 | SQLite | 项目、会话、消息、记忆、来源关系和任务状态等业务数据。 |
| 全文检索 | FTS5 | 关键词与全文检索。 |
| 向量检索 | sqlite-vec | 向量保存与相似度检索。 |
| 原始媒体 | 本地资源目录 | 保存图片、音频、视频等原文件，SQLite 管理元数据和引用。 |

## 为什么使用这一套

### TypeScript 作为主体

桌面界面、核心业务、工具和接入适配器使用同一门语言，便于个人持续维护。来自模型、接口和文件的数据仍需运行时校验，不能仅依赖 TypeScript 类型。

模型已经具备的理解、推理和工具选择能力由模型承担。具体业务方法通过 Skill、工具和工作流组织，运行核心负责通用执行行为。

### SQLite 作为主数据库

当前优先考虑本地使用与较低的部署负担。多个 Agent 通过 RainbowFish 的接口访问数据，数据库连接和事务由核心统一管理。

SQLite 是当前正式选择，不设定一个必须迁移 PostgreSQL 的时间点。如果后续出现持续写入拥堵，或明确需要多个后端实例共同访问中心数据库，再用实际需求与测量结果评估迁移。

PostgreSQL + pgvector 保留为未来候选，目前不同时实现两套存储。

### 业务记录、索引与媒体分工

业务记录和来源关系保存在普通表中；全文索引与向量用于检索。原始材料和记忆内容保留后，索引应能够重建。

Embedding 模型负责生成向量，sqlite-vec 负责存储和检索。向量模型、维度和版本需要能够识别，不能把不同向量空间当作同一套索引使用。

大体积原始媒体先放本地资源目录。资源标识与具体磁盘位置的映射由存储模块管理，外部调用方不依赖某台机器上的绝对路径。

## 实施时必须验证的地方

这些是后续相关 Feature 的验证输入，当前没有执行结果。

| 项目 | 需要验证的内容 |
| --- | --- |
| Windows 桌面安装包 | Electron 与原生模块版本匹配，打包后能打开数据库、加载 sqlite-vec、读写资源。 |
| 依赖与迁移 | 选择并锁定具体版本，验证迁移、升级和必要的恢复步骤；升级依据兼容性结果。 |
| 中文检索 | 验证分词、短词、代码标识符与中文混合文本，不能把启用 FTS5 当作已解决中文检索。 |
| 向量检索 | 用实际资料规模和目标设备验证延迟、召回与过滤行为；不预先承诺容量上限。 |
| SQLite 写入 | 保持写事务简短，模型调用和媒体处理在事务外进行；评估 WAL、忙等待及批量写入对交互的影响。 |
| 模型与工具 | 核对所选 Provider 的实际能力、工具调用和流式行为，按任务选择模型。 |

## 还没有选定的技术细节

- 模型 Provider、模型型号、Embedding 与重排模型。
- 中文分词策略和混合检索的排序方式。
- 工作流执行器、节点合同、暂停与恢复规则。React Flow 只确定编辑界面，不代替执行器。
- 桌面进程、后台任务和本地 HTTP 服务的具体运行方式。关闭窗口与退出服务的行为另行设计。
- 输入校验、日志、测试与界面样式等具体库，按工程基础需求选择。

Python 可在本地模型或媒体处理确有需求时补充。C++ 仅在已有库不能满足、且测量确认计算瓶颈时评估。二者都不属于当前必选运行环境。

## 官方资料

以下是后续核对能力与版本的入口，不代表项目已经完成集成。

- [Electron](https://www.electronjs.org/docs/latest/) 与 [electron-vite](https://electron-vite.org/guide/)
- [React Flow](https://reactflow.dev/)
- [AI SDK Core](https://ai-sdk.dev/docs/ai-sdk-core/overview)
- [Hono](https://hono.dev/docs/) 与 [MCP SDK](https://modelcontextprotocol.io/docs/sdk)
- [Drizzle SQLite](https://orm.drizzle.team/docs/get-started-sqlite)
- [SQLite](https://www.sqlite.org/docs.html)、[FTS5](https://www.sqlite.org/fts5.html)、[sqlite-vec](https://alexgarcia.xyz/sqlite-vec/)
