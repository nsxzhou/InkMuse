# InkMuse AGENTS

本文件定义 `InkMuse/` 根目录及其子目录下的默认工作规范，供人类开发者与 AI agent 共同遵循。

目标只有一个：在保持现有架构与产品行为稳定的前提下，让新增代码继续沿用项目已经形成的主流风格，而不是引入新的“个人流派”。

## 1. 作用域与仓库边界

- 本仓库是父仓库，`frontend/` 与 `backend/` 通过 Git submodule 管理。
- 父仓库主要负责总览文档、运行说明、子模块指针与跨仓约定；实际业务代码主要位于 `frontend/` 和 `backend/`。
- 修改子模块内容时，要把它视为进入对应子仓库工作：子模块内提交业务改动，父仓库只提交 submodule 指针变更。
- 本文件当前是仓库级默认规范。若未来在 `frontend/` 或 `backend/` 内新增更深层 `AGENTS.md`，以更深层文件为准。

## 2. 总体原则

- 先遵循现有模式，再考虑抽象或重构；没有明显收益时不要创造新范式。
- 优先最小改动面。新代码应尽量贴近所在模块现有命名、目录、依赖方向与错误处理方式。
- 共享契约只保留一个真相源，不允许前后端各写一份等价定义。
- 注释只解释边界、约束、原因或第三方兼容点，不解释显而易见的代码。
- 零星历史偏差不自动视为规范。若某个写法只在个别文件出现，而与仓库主流不一致，应视为例外。

## 3. 父仓库协作规则

- 克隆、更新与提交默认按 submodule 工作流处理：
  `git clone --recurse-submodules`
  `git submodule update --init --recursive`
- 父仓库提交应聚焦于：
  文档、运行说明、脚本、设计资料、规范文件、submodule 指针更新。
- 子模块提交应聚焦于：
  前端或后端业务代码、测试、构建配置、局部 README。
- 不要在父仓库 README 或规范文件中声称某个规则已经在子模块内“强制落地”，除非对应子模块中确实已有实现或文档支撑。

## 4. 通用编码规则

- 命名优先语义清晰，不使用难懂缩写；领域对象、用例、控制器、UI 面板等名称要直接反映职责。
- 新增文件应优先放入现有目录分层，不新增语义重复的目录。
- 保持“入口收敛、细节下沉”：
  页面或 handler 做参数接入与编排，复杂逻辑下沉到 hooks、service、helper 或 use case。
- 任何会被多处复用的规则、错误文案、查询 key、schema、派生逻辑，应抽到集中位置，而不是在页面或 handler 内重复拼装。
- 生成文件不要手改；需要调整生成结果时，回到生成源头或生成流程本身修改。

## 5. 前端规范

### 5.1 技术与工具基线

- 前端使用 React 18、TypeScript、Vite、TanStack Query、Tailwind CSS、Zod。
- TypeScript 以 `strict` 为基线，并开启 `noUnusedLocals`、`noUnusedParameters`、`noFallthroughCasesInSwitch`。
- 导入路径优先使用 `@/` 指向 `frontend/src`，避免深层相对路径穿透。
- ESLint 采用 zero-warning 策略；提交前不得留下 warning 级问题。

### 5.2 格式与语法风格

- 主流格式是：
  单引号、无分号、ES module、尾随逗号按现有 formatter 结果保留。
- 默认使用命名导出，避免无必要的 `export default`。
- 类型优先使用 `type`；仅在确有扩展或声明合并需求时再使用 `interface`。
- 导入顺序遵循现有主流模式：
  React 与第三方依赖在前，项目内 `@/` 导入在后，`type` 导入尽量与值导入分开或就地显式标注。
- 个别历史文件若使用双引号或分号，不视为仓库标准；新增或修改代码应回到主流风格。

### 5.3 文件命名与目录组织

- 文件名统一使用 kebab-case，例如：
  `project-workbench-page.tsx`
  `use-stream-task.ts`
  `query-invalidation.ts`
- 页面文件放在 `frontend/src/pages/`。
- 业务模块放在 `frontend/src/features/<domain>/`。
- 跨业务共享能力放在 `frontend/src/shared/`，其中：
  `shared/api/` 放 API 调用与共享类型边界
  `shared/ui/` 放基础 UI 组件
  `shared/lib/` 放通用工具函数
  `shared/config/` 放环境配置
- 应用级装配放在 `frontend/src/app/`，例如路由、Query Client、全局样式。
- 页面拆分优先沿用已有后缀模式：
  `*.hooks.ts`
  `*.screens.tsx`
  `*.test.tsx`

### 5.4 React 组件与 Hook 约定

- 统一使用函数组件和 Hooks。
- 页面组件负责路由参数、主查询、主 mutation、视图装配；复杂状态机和副作用应拆到自定义 hook。
- 复杂业务面板可以在 feature 内进一步拆分为：
  `components/`
  `lib/`
  `schemas/`
  `hooks` 风格文件
- 异步服务端状态统一优先使用 TanStack Query，不要在组件里手写重复的请求缓存逻辑。
- 流式任务统一封装为 hook 或 controller，避免把 SSE 生命周期直接散落在页面 JSX 中。
- 需要派生视图数据时，优先用纯函数或 `useMemo`，不要把可推导数据再存成 state。

### 5.5 类型、契约与 Schema 规则

- 后端生成的 `frontend/src/shared/api/generated/contracts.ts` 是共享 HTTP DTO、枚举、结构化 schema 的真相源，禁止手改。
- 前端对稳定共享协议的消费，优先直接引用 generated contracts 中的类型或 Zod schema。
- `frontend/src/shared/api/types.ts` 只保留 generated type 的 re-export 和少量前端增强类型，不重复手写后端已经稳定定义的 DTO。
- 页面表单、纯 UI 状态、局部 view-model 可以手写，但不得反向变成共享契约真相源。
- JSON 接口错误统一围绕 `error` 与可选 `error_code` 处理；用户可见错误文案优先走统一错误映射函数。
- 查询 key 统一集中定义在 `shared/api/queries.ts` 一类位置，保持 `as const` 风格，避免页面内散写字符串数组。
- 结构化表单和结构化资产优先使用 Zod schema，并延续当前做法：
  `extend()`
  `strict()`
  `infer`
  默认值与自定义 `superRefine()`

### 5.6 UI 与样式规则

- 设计语言延续 `scheme-05-minimal`：
  冷灰配色、Inter 字体、无阴影、无渐变、轻字重标题、过渡时间统一偏短。
- 样式优先复用已有语义 token：
  `background`
  `foreground`
  `muted`
  `border`
  `success`
  `warning`
  `danger`
- 优先使用 Tailwind utility 组合与共享基础组件，不要为一次性样式过早抽象复杂 class 工厂。
- 全局样式仅保留 token、基础排版、全局可访问性与第三方组件修饰；业务视觉细节优先留在组件层。
- 动画只保留少量有目的的过渡，不引入与现有极简风格冲突的大量视觉特效。

### 5.7 前端注释与例外

- JSX 注释主要用于分区标识或复杂布局导航。
- 对第三方库类型缺口、React Refresh 限制、编辑器存储 hack 等情况，可以写局部注释或最小范围 lint disable。
- `eslint-disable` 必须是局部、带明确理由的例外，不能作为常规开发手段。

### 5.8 前端测试

- 测试文件与被测模块就近放置，命名使用 `*.test.ts` 或 `*.test.tsx`。
- 优先覆盖纯逻辑模块、API 客户端、hook 行为、关键页面状态流转。
- 新增测试时优先延续 Vitest + Testing Library 的现有写法，不引入并行的测试范式。

## 6. 后端规范

### 6.1 技术与分层基线

- 后端使用 Go 1.24、Hertz、PostgreSQL。
- 目录分层保持稳定：
  `internal/domain/` 放领域模型与仓储接口
  `internal/service/` 放应用用例与业务编排
  `internal/infra/` 放 HTTP、LLM、存储、日志等基础设施实现
  `api/http/` 放路由注册
  `cmd/` 放可执行入口与工具
  `pkg/` 放可跨内部模块复用的稳定包
- 新能力优先进入既有分层；不要把 service 逻辑塞进 handler，也不要把存储细节泄漏到 domain。

### 6.2 命名与构造模式

- 包名保持小写、简短、领域化。
- 应用边界接口统一命名为 `UseCase`。
- 依赖注入结构体统一优先命名为 `Dependencies`。
- 构造函数统一优先使用：
  `NewUseCase`
  `NewService`
  `NewXxxHandler`
  `NewXxxRepository`
- 领域实体使用语义明确的名词；校验方法统一为 `Validate() error`。

### 6.3 输入清洗、校验与错误处理

- service 层入口先做字符串清洗，再做 UUID 与业务校验；主流做法是广泛使用 `strings.TrimSpace`。
- UUID 校验统一复用 `ValidateUUID`，不要各处手写一套。
- 非法输入统一包装为 `WrapInvalidInput(...)`。
- 仓储层错误统一通过 `TranslateStorageError(...)` 映射到 service 语义错误。
- handler 只负责：
  绑定参数
  调用 use case
  写回 HTTP 响应
  映射错误码
- HTTP 错误响应统一输出：
  `error`
  `error_code`
- 不要把数据库错误、基础设施错误原样暴露给前端。

### 6.4 实体创建与时间规则

- 新建实体默认使用 `uuid.NewString()` 生成主键。
- 时间字段统一使用 `time.Now().UTC()`。
- `CreatedAt` 与 `UpdatedAt` 必须保持 UTC 语义。
- 持久化前先校验实体；不要把无效对象直接写入仓储层。

### 6.5 注释与文档风格

- 导出类型、接口、构造函数、关键行为应写 GoDoc。
- 当前仓库以中文注释为主，可混用必要英文术语以避免歧义。
- 注释重点写职责、边界、失败语义、兼容策略，不要逐行翻译实现。
- 对跨层约束、超时策略、生成链路、迁移策略等关键设计，应在代码旁留下简短但明确的说明。

### 6.6 HTTP、契约与共享类型规则

- 后端是共享协议的真相源。稳定 DTO、枚举、结构化 seed/schema 应先在后端定义，再通过 `go run ./cmd/contractgen` 生成给前端。
- 任何共享字段变更，都应同时考虑：
  handler 请求绑定
  响应结构
  generated contracts
  前端消费层
- SSE 的 `done` / `result` / `error` 终态事件应与共享 schema 对齐；`content` 增量事件继续保持纯文本块语义。

### 6.7 后端测试

- 测试与实现文件就近放置，使用标准 `*_test.go`。
- 优先保持分层测试：
  domain model test
  service/use case test
  repository test
  handler integration test
- 测试辅助函数应语义明确、复用性强，例如构造 project、asset、chapter 的 helper。
- 涉及迁移、契约生成、LLM 适配、错误映射的改动，需要补足对应层级的回归测试。

## 7. 共享契约与生成代码规则

- `frontend/src/shared/api/generated/contracts.ts` 由后端 `contractgen` 生成，禁止手工编辑。
- 当前共享协议以“后端稳定定义 -> 生成到前端 -> 前端直接消费”为主路径。
- 如果某个字段已经进入后端稳定 DTO，就不要再在前端另写一份同名同义接口类型。
- 前端本地增强类型只能服务 UI，不得替代共享协议本身。

## 8. 文档与设计资料

- 根 README、运行文档、子模块 README 分工不同；不要把实现细节无差别堆到所有文档。
- 设计资料、提示词目录、研究文档属于参考资产，不自动等同于运行时代码规范。
- 当代码与旧设计稿或旧方案文档不一致时，以当前代码与当前运行文档为准。

## 9. 完成前检查

- 新增代码是否放在正确层级与目录，而不是创造新的平行结构。
- 命名、导出方式、类型定义方式是否与相邻文件一致。
- 前端是否继续复用 generated contracts、query keys、共享 UI 和错误处理工具。
- 后端是否继续复用 `Dependencies` / `UseCase` / `ValidateUUID` / `WrapInvalidInput` / `TranslateStorageError` 等既有模式。
- 是否误把个别历史例外当成了新规范。
- 是否需要同步测试、文档或 submodule 指针。
