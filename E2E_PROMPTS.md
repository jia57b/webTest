# E2E Testing Prompts

## 索引

- [使用指南](#guide)
- [1. 根据代码生成端到端测试场景](#prompt-1)
- [2. 基于 PR diff 生成端到端测试场景](#prompt-2)
- [3. 基于 GitHub PR URL 自动拉取 diff 并生成端到端测试场景](#prompt-3)
- [4. 基于 GitHub PR URL 直接生成 Playwright 测试代码](#prompt-4)
- [5. 根据测试场景生成 Playwright 测试骨架](#prompt-5)
- [6. 根据测试场景生成 Playwright 测试代码](#prompt-6)
- [7. 运行测试并生成集成测试报告](#prompt-7)
- [8. 根据 Jira ticket 自动生成测试场景和测试用例（自动读取 Jira 版）](#prompt-8)
- [9. 根据 Jira ticket 自动生成测试场景和测试用例（手工输入版）](#prompt-9)
- [推荐使用顺序](#recommended-order)
- [使用建议](#usage-notes)

<a id="guide"></a>
## 使用指南

这份文档作为一份 Web 测试指导文档使用，目标是指导你从零开始完成下面这条链路：

1. 安装或接入所需 skill
2. 准备本地项目和输出目录
3. 使用 Prompt 生成 E2E 场景
4. 生成 Playwright 骨架
5. 生成 Playwright 测试代码
6. 运行测试并输出测试报告

### 1. 本项目自带的本地 skills

本项目已经自带一份本地 `skills/` 目录：

- `./skills/testing-qa`
- `./skills/test-automator`
- `./skills/playwright-skill`
- `./skills/differential-review`
- `./skills/verification-before-completion`
- `./skills/lint-and-validate`
- `./skills/jira-automation`
- `./skills/prompt-engineer`

建议把这份项目内 `./skills/` 视为唯一基线版本。

其中：

- 前 7 个是这份文档里实际测试工作流会直接依赖的 skill
- `prompt-engineer` 主要用于优化和改写 Prompt，本身不是执行这些测试工作流的必需 skill

### 2. 安装前提

在开始前，建议先确认：

- 已经能打开并运行当前项目
- 本机已安装 Node.js
- 项目依赖可以正常安装
- 如果涉及 GitHub PR URL：
  - 已安装 `gh`
  - 已完成 `gh auth login`
- 如果要运行 Playwright：
  - 已安装 Playwright 依赖
  - 浏览器环境可用

#### 如何安装 `gh`

`gh` 是 GitHub CLI。  
当你要使用以下 Prompt 时，通常建议先安装并登录 `gh`：

- 基于 GitHub PR URL 自动拉取 diff 并生成端到端测试场景
- 基于 GitHub PR URL 直接生成 Playwright 测试代码

常见安装方式如下。

**macOS**

如果你使用 Homebrew：

```bash
brew install gh
```

安装后验证：

```bash
gh --version
```

#### 如何执行 `gh auth login`

安装完成后，执行：

```bash
gh auth login
```

常见交互流程：

1. 选择 `GitHub.com`
2. 选择认证协议，通常选 `HTTPS`
3. 选择登录方式，通常选浏览器登录
4. 按提示完成授权

登录完成后验证：

```bash
gh auth status
```

如果输出里能看到你当前登录的 GitHub 账号，说明已经可以用于：

- `gh pr view <PR_URL>`
- `gh pr diff <PR_URL>`

如果仓库是私有仓库，还需要确认当前 GitHub 账号对该仓库有访问权限。

#### 如何安装 Playwright 依赖

当你要使用以下 Prompt 时，通常需要先安装 Playwright 依赖：

- 根据测试场景生成 Playwright 测试骨架
- 根据测试场景生成 Playwright 测试代码
- 运行测试并生成集成测试报告
- 基于 GitHub PR URL 直接生成 Playwright 测试代码

查看 ./skills/playwright-skill/SKILL.md进行安装。

#### 验证是否安装成功

```bash
npx playwright --version
```

### 3. 如何安装或接入这些 skill

#### 方式 A：优先直接使用项目内 skills

如果你的工具支持从当前项目读取本地 `./skills/`，这是最推荐的方式：

- 保持仓库里的 `./skills/` 不动
- 在项目根目录启动你的 AI 工具
- 让工具直接读取当前项目中的本地 skills

#### 方式 B：复制或软链接到工具自己的全局 skills 目录

如果目标工具不会自动读取项目内 `./skills/`，就把这些 skill 接入全局目录。

示例命令：

```bash
ln -s /absolute/path/to/project/skills/testing-qa <TOOL_SKILLS_DIR>/testing-qa
ln -s /absolute/path/to/project/skills/test-automator <TOOL_SKILLS_DIR>/test-automator
ln -s /absolute/path/to/project/skills/playwright-skill <TOOL_SKILLS_DIR>/playwright-skill
ln -s /absolute/path/to/project/skills/differential-review <TOOL_SKILLS_DIR>/differential-review
ln -s /absolute/path/to/project/skills/verification-before-completion <TOOL_SKILLS_DIR>/verification-before-completion
ln -s /absolute/path/to/project/skills/lint-and-validate <TOOL_SKILLS_DIR>/lint-and-validate
ln -s /absolute/path/to/project/skills/prompt-engineer <TOOL_SKILLS_DIR>/prompt-engineer
```

如果不想用软链接，也可以复制：

```bash
cp -R /absolute/path/to/project/skills/testing-qa <TOOL_SKILLS_DIR>/
cp -R /absolute/path/to/project/skills/test-automator <TOOL_SKILLS_DIR>/
cp -R /absolute/path/to/project/skills/playwright-skill <TOOL_SKILLS_DIR>/
cp -R /absolute/path/to/project/skills/differential-review <TOOL_SKILLS_DIR>/
cp -R /absolute/path/to/project/skills/verification-before-completion <TOOL_SKILLS_DIR>/
cp -R /absolute/path/to/project/skills/lint-and-validate <TOOL_SKILLS_DIR>/
cp -R /absolute/path/to/project/skills/prompt-engineer <TOOL_SKILLS_DIR>/
```

#### 方式 C：如果工具不支持 skills 目录

如果目标工具不支持 skills 目录或 skill 包机制，那就不要强行安装 skill。

这时应该把 skill 的工作流迁移成：

- rules
- custom instructions
- prompt files
- AGENTS.md
- system prompt

这份文档里的 Prompt 仍然可以继续使用，只需要：

1. 删掉 `Use @skill-name ...`
2. 保留任务本体
3. 把 skill 名换成职责描述

例如：

```text
请按以下工作流执行：
- 测试策略分析
- PR diff 影响分析
- E2E 场景设计
- Playwright 骨架生成
- Playwright 测试实现
- 测试执行验证与报告输出
```

### 4. 最小推荐 skill 组合

#### 只生成 E2E 场景

- `testing-qa`
- `test-automator`

#### 基于 PR diff 生成 E2E 场景

- `testing-qa`
- `test-automator`
- `differential-review`

#### 生成 Playwright 骨架和代码

- `playwright-skill`
- `test-automator`

#### 跑测试并出报告

- `playwright-skill`
- `verification-before-completion`
- `lint-and-validate`

### 5. 推荐的本地目录结构

建议在项目中统一使用下面的输出目录：

```text
docs/testing/
tests/e2e/
reports/e2e/
```

例如：

- 场景文档：
  - `docs/testing/e2e-scenarios.md`
  - `docs/testing/pr-123-e2e-scenarios.md`
- Playwright 测试：
  - `tests/e2e/specs/*.spec.ts`
- 测试报告：
  - `reports/e2e/latest-report.md`
  - `reports/e2e/pr-123-report.md`

### 6. 使用 Prompt 的顺序

本文件中的 Prompt 建议按下面顺序使用：

1. 如果是全量分析，先用“根据代码生成端到端测试场景”
2. 如果是本地 diff，先用“基于 PR diff 生成端到端测试场景”
3. 如果是 GitHub PR URL，先用“基于 GitHub PR URL 自动拉取 diff 并生成端到端测试场景”
4. 如果要从 PR URL 一步直达代码，使用“基于 GitHub PR URL 直接生成 Playwright 测试代码”
5. 再用“根据测试场景生成 Playwright 测试骨架”
6. 再用“根据测试场景生成 Playwright 测试代码”
7. 最后用“运行测试并生成集成测试报告”

### 7. 如何填写 Prompt 里的占位符

每份 Prompt 里都有占位符，使用前需要替换成真实值。

最常见的有：

- `<PROJECT_PATH>`
- `<SCENARIO_OUTPUT_FILE>`
- `<PLAYWRIGHT_TEST_DIR>`
- `<PLAYWRIGHT_OUTPUT_DIR>`
- `<REPORT_OUTPUT_FILE>`
- `<PR_URL>`
- `<PR_DIFF_OR_COMMAND>`
- `<BASE_URL_OR_ENV>`

示例：

```text
<PROJECT_PATH> -> /Users/57block/Jia/webTest/wcp-ui
<SCENARIO_OUTPUT_FILE> -> /Users/57block/Jia/webTest/docs/testing/pr-123-e2e-scenarios.md
<PLAYWRIGHT_TEST_DIR> -> /Users/57block/Jia/webTest/wcp-ui/tests/e2e
<REPORT_OUTPUT_FILE> -> /Users/57block/Jia/webTest/reports/e2e/pr-123-report.md
```

### 8. 使用 Prompt 时的统一要求

无论使用哪一份 Prompt，都建议坚持以下规则：

1. 结果必须写到本地文件，不只是显示在聊天里
2. 只输出代码或 diff 有证据支持的内容
3. 优先覆盖高价值路径：
   - 登录
   - 权限
   - 导航
   - 表单提交
   - 成功/失败提示
   - 删除/支付等高风险动作
4. 生成 Playwright 时优先用稳定 locator
5. 没有真实执行结果，不要声称测试通过

### 9. 一句话总结

把这份文档当成一套 Web 测试工作流：

- 先安装或接入 skill
- 再选对 Prompt
- 再替换占位符
- 再把输出落到本地文件
- 最后执行验证并产出测试报告


本文档包含 9 份可直接复用的 Prompt：

1. 根据代码生成端到端测试场景
2. 基于 PR diff 生成端到端测试场景
3. 基于 GitHub PR URL 自动拉取 diff 并生成端到端测试场景
4. 基于 GitHub PR URL 直接生成 Playwright 测试代码
5. 根据测试场景生成 Playwright 测试骨架
6. 根据测试场景生成 Playwright 测试代码
7. 运行测试并生成集成测试报告
8. 根据 Jira ticket 自动生成测试场景和测试用例（自动读取 Jira 版）
9. 根据 Jira ticket 自动生成测试场景和测试用例（手工输入版）

<a id="prompt-1"></a>
## 1. 根据代码生成端到端测试场景

说明：

- 这一份 Prompt 需要在**能够访问前端代码**的前提下使用。请先获取前端代码到本地。

```text
Use @testing-qa and @test-automator to analyze this web application and generate high-value end-to-end test scenarios from code.

任务目标：
根据代码而不是猜测，提炼可执行、可追溯、适合后续 Playwright 落地的 E2E 测试场景。

项目上下文：
- 项目路径：<PROJECT_PATH>
- 技术栈：<React/Vue/Next/Nuxt/...>
- 路由方式：<react-router / next router / vue-router / file-based router>
- 鉴权方式：<token/cookie/session/SSO>
- 输出语言：中文
- 只生成测试场景，不生成测试代码
- 场景输出文件：<SCENARIO_OUTPUT_FILE>

请重点分析这些代码：
- 路由文件：<ROUTE_FILES>
- 鉴权/权限控制文件：<AUTH_FILES>
- 页面文件：<PAGE_FILES>
- service/api/loader/hooks 文件：<SERVICE_FILES>
- 表单/校验相关文件：<FORM_FILES>

分析要求：
1. 先识别页面入口、URL、受保护路由、公开路由。
2. 识别页面前置条件：
   - 登录态
   - 用户角色/权限
   - loader 数据
   - feature flag
   - 依赖接口返回
3. 识别关键用户操作：
   - 访问页面
   - 提交表单
   - 切换 tab
   - 弹窗确认
   - 跳转
   - 上传
   - 筛选/搜索/分页
4. 识别分支和异常：
   - required 校验
   - if/else 分支
   - 成功提示
   - 错误提示
   - 接口失败
   - 无数据/空状态
   - 重定向
5. 只输出代码中有证据支持的场景，不要臆造功能。
6. 优先输出高价值场景，不要穷举低价值点击。
7. 将最终结果保存到本地 `<SCENARIO_OUTPUT_FILE>`，格式为 Markdown。

输出格式必须严格如下：

# 应用概览
- 主要页面：
- 主要受保护页面：
- 主要业务高风险点：
- 推荐优先覆盖模块：

# 页面级 E2E 测试场景

## 页面：<页面名称>
- URL：
- 来源文件：
- 前置条件：
- 关键依赖：
- 建议优先级：P0 / P1 / P2

### 场景 1：<标题>
- 类型：访问控制 / 成功路径 / 校验 / 分支 / 异常 / 回归
- 前置条件：
- 操作步骤：
- 预期结果：
- 测试数据需求：
- 来源文件：
- 触发依据：

### 场景 2：<标题>
- 类型：
- 前置条件：
- 操作步骤：
- 预期结果：
- 测试数据需求：
- 来源文件：
- 触发依据：

# 跨页面关键回归场景
- 场景：
- 涉及页面：
- 风险说明：
- 来源文件：

# 缺失信息/待确认项
- 项目：
- 影响：

补充要求：
- 每个受保护页面至少考虑：
  - 未登录访问
  - 已登录正常访问
  - 无权限访问
- 每个有表单的页面至少考虑：
  - 必填校验
  - 成功提交
  - 接口失败
- 每个有异步加载的页面至少考虑：
  - 加载成功
  - 空状态
  - 加载失败
- 不要写 Playwright 代码，只输出测试场景。
- 最终除了在对话中展示摘要，还必须把完整结果写入 `<SCENARIO_OUTPUT_FILE>`。
```

<a id="prompt-2"></a>
## 2. 基于 PR diff 生成端到端测试场景

```text
Use @differential-review, @test-automator, and @testing-qa to analyze a PR diff and generate the end-to-end test scenarios that should be added or updated because of this change.

任务目标：
基于 PR diff，而不是全量代码扫描，识别这次改动真正影响到的用户路径、页面状态、权限行为、表单分支和跨页面回归风险，并输出需要新增、修改或回归执行的 E2E 测试场景。

项目上下文：
- 项目路径：<PROJECT_PATH>
- PR 基线分支：<BASE_BRANCH>
- 对比分支或提交：<HEAD_REF>
- diff 来源：<git diff BASE...HEAD / PR URL / patch 内容>
- 技术栈：<React/Vue/Next/Nuxt/...>
- 路由方式：<react-router / next router / vue-router / file-based router>
- 鉴权方式：<token/cookie/session/SSO>
- 输出语言：中文
- 只生成测试场景，不生成测试代码
- 场景输出文件：<SCENARIO_OUTPUT_FILE>

输入材料：
- PR diff：<PR_DIFF_OR_COMMAND>
- 变更文件列表：<CHANGED_FILES>
- 如果需要补上下文，请读取：
  - 路由文件：<ROUTE_FILES>
  - 鉴权文件：<AUTH_FILES>
  - 被修改页面或其调用方：<PAGE_FILES>
  - 相关 service/api/loader/hooks：<SERVICE_FILES>

分析要求：
1. 先基于 diff 识别本次改动涉及的文件类别：
   - source
   - test
   - config
   - auth/security-sensitive
   - route
   - api/service
   - UI/page/component
2. 对每个改动文件判断它是否会影响：
   - 页面入口或导航
   - 权限控制
   - 用户输入/表单提交
   - 数据加载与渲染
   - 成功/失败提示
   - 跨页面回归路径
3. 不要把所有改动都升格成 E2E 场景，只输出与用户可感知行为相关的改动。
4. 对纯重构、样式、小型文案改动，要判断是否只需要 smoke regression，而不是生成完整新场景。
5. 如遇高风险改动，必须提升优先级：
   - auth
   - role/permission
   - route guards
   - payment / billing
   - destructive action
   - external API integration
   - validation removal or logic change
6. 每个场景都必须明确指出：
   - 这是“新增场景”、“修改现有场景”还是“建议回归重跑”
   - 它直接对应哪段 diff
7. 只输出代码和 diff 有证据支持的场景，不要臆造功能。
8. 将最终结果保存到本地 `<SCENARIO_OUTPUT_FILE>`，格式为 Markdown。

输出格式必须严格如下：

# PR 变更概览
- 基线：
- 变更范围：
- 主要受影响模块：
- 风险等级：低 / 中 / 高
- 建议优先验证方向：

# 基于 PR diff 的 E2E 测试场景

## 模块：<模块名称>
- 变更文件：
- 变更类型：新增 / 修改 / 删除 / 重构
- 用户影响判断：
- 建议优先级：P0 / P1 / P2

### 场景 1：<标题>
- 场景动作：新增场景 / 修改现有场景 / 回归重跑
- 类型：访问控制 / 成功路径 / 校验 / 分支 / 异常 / 回归
- 前置条件：
- 操作步骤：
- 预期结果：
- 测试数据需求：
- 对应 diff 文件：
- 对应 diff 依据：
- 为什么需要这个场景：

### 场景 2：<标题>
- 场景动作：
- 类型：
- 前置条件：
- 操作步骤：
- 预期结果：
- 测试数据需求：
- 对应 diff 文件：
- 对应 diff 依据：
- 为什么需要这个场景：

# 建议更新的现有 E2E 用例
- 现有用例或模块：
- 建议修改点：
- 对应 diff 文件：
- 原因：

# 建议重跑的回归场景
- 场景：
- 涉及页面：
- 风险说明：
- 对应 diff 文件：

# 不需要新增 E2E 的改动
- 文件：
- 原因：

# 缺失信息/待确认项
- 项目：
- 影响：

补充要求：
- 如果 diff 涉及受保护路由、角色、重定向、导航，至少生成权限相关场景。
- 如果 diff 涉及表单字段、校验、提交 body、错误提示，至少生成表单相关场景。
- 如果 diff 涉及接口返回、空状态、列表渲染、分页筛选，至少生成数据渲染或交互场景。
- 如果 diff 涉及共享组件或基础 hook，要额外判断 blast radius，列出受影响的页面回归路径。
- 不要写 Playwright 代码，只输出测试场景和回归建议。
- 最终除了在对话中展示摘要，还必须把完整结果写入 `<SCENARIO_OUTPUT_FILE>`。
```

<a id="prompt-3"></a>
## 3. 基于 GitHub PR URL 自动拉取 diff 并生成端到端测试场景

```text
Use @differential-review, @test-automator, and @testing-qa to fetch a GitHub pull request diff from its URL, analyze the change set, and generate the end-to-end test scenarios that should be added, updated, or rerun.

任务目标：
输入一个 GitHub PR URL，让 AI 自动获取 PR 元信息和 diff，然后基于真实改动生成对应的 E2E 测试场景，而不是基于猜测或全量代码扫描。

输入：
- GitHub PR URL：<PR_URL>
- 本地项目路径：<PROJECT_PATH>
- 是否允许使用 gh CLI：<YES_OR_NO>
- 输出语言：中文
- 只生成测试场景，不生成测试代码
- 场景输出文件：<SCENARIO_OUTPUT_FILE>

可用工具优先级：
1. 优先使用 `gh pr view`、`gh pr diff`
2. 如果 `gh` 不可用，再尝试从 URL 提取：
   - repo owner
   - repo name
   - PR number
3. 如果无法直接获取完整 diff，至少获取：
   - 文件变更列表
   - PR 标题
   - PR 描述
   - 相关提交列表
4. 如果 PR diff 过大，按高风险模块优先分析，不要被大 diff 淹没

执行要求：
1. 从 PR URL 中提取：
   - 仓库
   - PR 编号
   - base branch
   - head branch
2. 自动拉取以下信息：
   - PR 标题
   - PR 描述
   - changed files
   - 完整 diff 或 patch
   - 如果可能，获取提交列表
3. 结合本地代码上下文补充分析：
   - 受影响路由
   - 鉴权逻辑
   - 被修改页面
   - service/api/hook
4. 只针对“用户可感知行为变化”生成 E2E 场景：
   - 页面导航变化
   - 权限变化
   - 表单校验变化
   - 提交逻辑变化
   - 渲染分支变化
   - 错误提示/成功提示变化
   - 跨页面回归路径
5. 对以下情况必须提高优先级：
   - auth / role / permission / route guards
   - billing / payment / destructive actions
   - validation removal
   - API contract change
   - shared component / shared hook / shared service
6. 如果 diff 中包含大量无关变更，要显式区分：
   - 需要新增 E2E 的改动
   - 只需更新现有 E2E 的改动
   - 只需回归重跑的改动
   - 不需要 E2E 的改动
7. 只输出有 diff 或代码证据支持的场景，不要臆造。
8. 将最终结果保存到本地 `<SCENARIO_OUTPUT_FILE>`，格式为 Markdown。

如果需要使用的命令示例：
- `gh pr view <PR_URL> --json number,title,body,baseRefName,headRefName,files`
- `gh pr diff <PR_URL>`

输出格式必须严格如下：

# GitHub PR 概览
- PR URL：
- 仓库：
- PR 编号：
- 标题：
- base：
- head：
- 变更文件数：
- 风险等级：低 / 中 / 高
- 建议优先验证方向：

# 变更文件摘要
- 文件：
  - 类型：
  - 是否用户可感知：
  - 是否需要 E2E：
  - 原因：

# 基于 PR URL + diff 的 E2E 测试场景

## 模块：<模块名称>
- 变更文件：
- 用户影响判断：
- 建议优先级：P0 / P1 / P2

### 场景 1：<标题>
- 场景动作：新增场景 / 修改现有场景 / 回归重跑
- 类型：访问控制 / 成功路径 / 校验 / 分支 / 异常 / 回归
- 前置条件：
- 操作步骤：
- 预期结果：
- 测试数据需求：
- 对应 PR 文件：
- 对应 diff 依据：
- 为什么需要这个场景：

### 场景 2：<标题>
- 场景动作：
- 类型：
- 前置条件：
- 操作步骤：
- 预期结果：
- 测试数据需求：
- 对应 PR 文件：
- 对应 diff 依据：
- 为什么需要这个场景：

# 建议更新的现有 E2E 用例
- 现有用例或模块：
- 建议修改点：
- 对应 PR 文件：
- 原因：

# 建议重跑的回归场景
- 场景：
- 涉及页面：
- 风险说明：
- 对应 PR 文件：

# 不需要新增 E2E 的改动
- 文件：
- 原因：

# 缺失信息/阻塞项
- 项目：
- 影响：

补充要求：
- 若 `gh` 命令不可用，要明确说明，并给出降级分析结果。
- 若 diff 被截断，要注明覆盖范围不足。
- 若 PR 只涉及文档、样式、注释、测试文件本身，应说明为什么不需要新增 E2E。
- 不要写 Playwright 代码，只输出场景和回归建议。
- 最终除了在对话中展示摘要，还必须把完整结果写入 `<SCENARIO_OUTPUT_FILE>`。
```

<a id="prompt-4"></a>
## 4. 基于 GitHub PR URL 直接生成 Playwright 测试代码

```text
Use @differential-review, @test-automator, @testing-qa, and @playwright-skill to fetch a GitHub PR from its URL, analyze the diff, derive the required E2E scenarios, and directly implement Playwright test code in the local repository.

任务目标：
输入一个 GitHub PR URL，让 AI 自动获取 PR diff，识别需要新增或更新的 E2E 场景，并直接在本地项目中生成对应的 Playwright 测试代码。

输入：
- GitHub PR URL：<PR_URL>
- 本地项目路径：<PROJECT_PATH>
- 是否允许使用 gh CLI：<YES_OR_NO>
- Playwright 测试目录：<PLAYWRIGHT_TEST_DIR>
- Playwright 配置路径：<PLAYWRIGHT_CONFIG_PATH>
- baseURL 或环境变量名：<BASE_URL_OR_ENV>
- 登录方式说明：<AUTH_SETUP_INFO>
- 测试账号来源：<TEST_ACCOUNT_INFO>
- 若需要中间产物，场景文档输出文件：<SCENARIO_OUTPUT_FILE>

执行目标：
1. 从 PR URL 自动拉取：
   - PR 标题
   - PR 描述
   - 变更文件列表
   - diff / patch
2. 基于 diff 判断哪些改动需要：
   - 新增 Playwright 用例
   - 修改现有 Playwright 用例
   - 仅重跑回归
3. 先把分析得到的测试场景保存到 `<SCENARIO_OUTPUT_FILE>`。
4. 再根据这些场景，直接在本地 `<PLAYWRIGHT_TEST_DIR>` 中创建或更新测试代码。
5. 如果项目还没有 Playwright 基础结构，则补齐最小可用结构，但不要破坏现有项目约定。

代码生成要求：
1. 只为 PR 改动影响到的用户行为生成测试，不要全量重写测试。
2. 优先覆盖 P0 / P1 场景。
3. 必须优先使用稳定 locator：
   - getByRole
   - getByLabel
   - getByText
   - data-testid
4. 必须处理：
   - 路由跳转
   - 登录态
   - 权限控制
   - 表单校验
   - 成功提示
   - 错误提示
   - confirm dialog
5. 对异步页面必须使用可靠等待，禁止无意义 sleep。
6. 如果缺失测试数据、环境变量、稳定选择器、登录手段或必要上下文，不要硬写假代码，要明确标记 blocker。
7. 生成的代码必须直接保存到本地目标文件，而不是只在对话里展示。

输出格式：

# PR 到 Playwright 实现计划
- PR URL：
- 主要变更模块：
- 需要新增的测试：
- 需要修改的测试：
- 仅建议回归重跑的测试：

# 本地变更文件
- 文件：
  - 动作：新增 / 修改
  - 原因：

# 场景文档输出
- 文件路径：<SCENARIO_OUTPUT_FILE>

# 未落地的场景与阻塞原因
- 场景：
- 阻塞原因：
- 建议补充信息：

# 推荐执行命令
- 安装：
- 运行单测：
- 运行 E2E：

补充要求：
- 如果 `gh` 不可用，要明确说明，并使用降级方式处理。
- 如果 diff 过大，只优先实现高风险路径。
- 除了简要摘要外，不要把完整代码只停留在对话中，必须写到本地文件。
```

<a id="prompt-5"></a>
## 5. 根据测试场景生成 Playwright 测试骨架

说明：

- 这一份 Prompt 最好在**能够访问前端代码**的前提下使用。
- 原因是 Playwright 骨架通常需要结合真实项目结构来决定：
  - 路由组织
  - 页面对象拆分
  - fixture 设计
  - 现有测试目录
  - 配置文件位置
- 如果当前只能访问 Jira 或需求文档，而**不能访问前端代码**，这份 Prompt 仍可用于生成“模板级骨架草案”，但不适合直接生成项目可落地的最终骨架。

```text
Use @testing-qa and @playwright-skill to generate a Playwright test skeleton for this project based on the provided E2E scenarios.

任务目标：
根据已有测试场景，为当前项目生成“可扩展、可维护、可继续填充”的 Playwright 测试骨架，不要求一次写完所有断言细节，但要建立正确结构。

项目上下文：
- 项目路径：<PROJECT_PATH>
- 技术栈：<React/Vue/Next/...>
- 包管理器：<npm/yarn/pnpm>
- 目标测试框架：Playwright
- 输出语言：中文说明，代码用项目当前语言风格
- 已有测试场景文档：<SCENARIO_DOC_PATH>
- 是否已有 Playwright：<YES_OR_NO>
- 骨架输出根目录：<PLAYWRIGHT_OUTPUT_DIR>

请执行以下工作：
1. 检查当前项目是否已有 Playwright 配置。
2. 如果没有，则创建最小可用 Playwright 目录结构。
3. 如果已有，则在现有结构中补齐规范化骨架，不要破坏现有约定。
4. 根据测试场景，生成：
   - playwright.config
   - tests 目录结构
   - fixtures
   - page objects 或 page models
   - auth helper
   - test data helper
   - smoke spec skeleton
   - critical-path spec skeleton
5. baseURL、账号、环境变量必须参数化，不要硬编码。
6. 优先使用稳定 locator 策略：
   - getByRole
   - getByLabel
   - getByText
   - data-testid
7. 不要直接写脆弱的 CSS 选择器，除非代码里没有更稳定标识。
8. 骨架里允许保留 TODO，但每个 TODO 必须具体，不允许空泛。
9. 所有生成的骨架文件必须直接保存到本地 `<PLAYWRIGHT_OUTPUT_DIR>`。

输出要求：
1. 先输出建议的目录结构。
2. 再输出每个文件的作用。
3. 再输出骨架代码。
4. 最后输出：
   - 如何运行
   - 还缺哪些信息
   - 下一步最适合补哪几个测试

请优先生成以下文件类型：
- <PLAYWRIGHT_CONFIG_PATH>
- <TESTS_DIR>/fixtures/*.ts
- <TESTS_DIR>/pages/*.ts
- <TESTS_DIR>/specs/*.spec.ts
- <TESTS_DIR>/utils/*.ts

如果项目中缺少适合 E2E 的稳定选择器，请单独列出建议补充的 data-testid 清单。
- 除了说明外，必须把实际骨架文件写到本地。
```

<a id="prompt-6"></a>
## 6. 根据测试场景生成 Playwright 测试代码

说明：

- 这一份 Prompt 基本要求**能够访问前端代码**后再使用。
- 原因是可运行的 Playwright 测试代码通常依赖真实前端信息，例如：
  - 页面结构
  - 路由
  - 文案
  - 表单字段
  - 稳定 locator
  - 登录方式
  - 现有测试基建
- 如果当前只有 Jira ticket、需求说明或测试场景，而**不能访问前端代码**，这份 Prompt 只能用于生成：
  - Playwright 伪代码
  - 测试模板
  - 待补 locator 的草案
- 不建议在无法访问前端代码的情况下，直接要求它生成“可运行、可落地”的最终 Playwright 测试代码。

```text
Use @playwright-skill and @test-automator to implement Playwright test code from the provided E2E scenarios.

任务目标：
把测试场景转成实际可运行的 Playwright 测试代码，优先覆盖 P0 / P1 场景，并保持测试稳定、可维护、可读。

输入：
- 项目路径：<PROJECT_PATH>
- 测试场景文档：<SCENARIO_DOC_PATH>
- Playwright 骨架路径：<PLAYWRIGHT_TEST_DIR>
- 目标页面或模块：<TARGET_MODULES>
- 测试环境 URL：<BASE_URL or AUTO_DETECT>
- 登录方式说明：<AUTH_SETUP_INFO>
- 测试账号来源：<TEST_ACCOUNT_INFO>
- 代码输出目录：<PLAYWRIGHT_TEST_DIR>

实现要求：
1. 只根据测试场景和代码证据生成测试，不要臆造 DOM 结构。
2. 优先实现 P0 和 P1 场景。
3. 每个场景必须映射到具体 spec。
4. 合理抽取：
   - page object
   - fixture
   - helper
   - test data builder
5. 每个测试必须包含明确断言，不能只有点击流程。
6. 优先使用稳定定位方式：
   - getByRole
   - getByLabel
   - getByText
   - data-testid
7. 对异步页面要有可靠等待：
   - waitForURL
   - expect(locator).toBeVisible()
   - API/mock 完成后再断言
8. 对以下情况要显式处理：
   - 登录态初始化
   - 跳转
   - 错误 toast / notification
   - confirm dialog
   - 表单必填校验
   - 接口失败
9. 如果某个场景因缺失测试数据、缺少稳定选择器、依赖第三方登录、依赖外部系统而无法落地，请不要硬写假代码，要明确标记 blocker。
10. 所有新增或修改的测试代码必须直接写入本地 `<PLAYWRIGHT_TEST_DIR>`。

输出格式：
1. 测试实现计划
2. 变更文件列表
3. 每个文件的完整代码
4. 未实现的场景及原因
5. 推荐执行命令

实现风格要求：
- 使用 test.describe / test.step
- 断言要具体
- 避免重复代码
- 不要写无意义 sleep
- 不要把业务逻辑硬编码进测试
- 不要把凭证明文写进代码
- 除了摘要说明外，不要只展示代码，必须把代码写入本地文件。
```

<a id="prompt-7"></a>
## 7. 运行测试并生成集成测试报告

```text
Use @playwright-skill, @verification-before-completion, and @lint-and-validate to run the Playwright tests, verify the real results, and produce an integration test report.

任务目标：
实际运行测试，基于真实命令输出生成一份集成测试报告。禁止在没有新鲜验证结果的情况下声称“测试通过”。

项目上下文：
- 项目路径：<PROJECT_PATH>
- 测试命令：<TEST_COMMAND>
- 测试范围：<TEST_SCOPE>
- 可选验证命令：
  - lint：<LINT_COMMAND>
  - typecheck：<TYPECHECK_COMMAND>
  - build：<BUILD_COMMAND>
- 测试报告目录：<REPORT_DIR>
- 截图/trace/video 目录：<ARTIFACT_DIR>
- 测试报告输出文件：<REPORT_OUTPUT_FILE>

严格要求：
1. 必须先运行真实命令，再写报告。
2. 如果提供了 `<TEST_COMMAND>`，直接优先使用该命令。
3. 如果没有提供 `<TEST_COMMAND>`，但提供了 `<TEST_SCOPE>`，则必须先自动识别项目中的真实测试命令，再按 `<TEST_SCOPE>` 选择执行范围。
4. `<TEST_SCOPE>` 可以是：
   - 全部测试
   - 冒烟测试
   - 指定目录
   - 指定模块
   - 指定 spec
   - 登录相关测试
   - 结账相关测试
   - 其他功能子集
5. 如果既没有 `<TEST_COMMAND>`，也没有 `<TEST_SCOPE>`，要先明确说明缺少执行入口，不能伪造测试结果。
6. 报告只能基于本次运行结果，不允许引用旧结果。
7. 如果测试失败，不要声称通过，要如实说明失败数量、失败原因和证据。
8. 如果命令无法运行，要说明阻塞原因。
9. 如有可能，请补充：
   - 失败用例截图
   - trace 路径
   - video 路径
   - 控制台错误
   - 网络错误
10. 先给出事实，再给出判断。

执行顺序：
1. 检查测试环境是否可运行。
2. 如果 `<TEST_COMMAND>` 为空，则先自动识别测试命令。识别顺序：
   - package.json scripts
   - README / docs / CONTRIBUTING
   - Playwright / Cypress / Jest / Vitest 配置文件
   - CI 配置文件
3. 如果识别出多个候选命令：
   - 列出候选项
   - 说明区别
   - 默认选择最适合本地验证的一条
4. 根据 `<TEST_SCOPE>` 决定运行全部测试还是指定测试子集。
5. 运行 lint / typecheck / build（如果这些是本项目验证链的一部分）。
6. 运行最终确定的测试命令。
7. 读取并总结真实输出。
8. 生成测试报告。
9. 将完整报告写入本地 `<REPORT_OUTPUT_FILE>`。

输出格式必须严格如下：

# 集成测试报告

## 1. 执行摘要
- 执行时间：
- 项目路径：
- 输入的测试命令：
- 输入的测试范围：
- 最终执行命令：
- 是否执行成功：是 / 否
- 总体结论：通过 / 部分通过 / 失败 / 阻塞

## 2. 验证证据
- lint 结果：
- typecheck 结果：
- build 结果：
- Playwright 结果：
- 命令退出码：
- 关键原始输出摘要：

## 3. 测试统计
- 总用例数：
- 通过：
- 失败：
- 跳过：
- flaky：
- 未执行：

## 4. 失败明细
### 失败用例：<名称>
- 所属文件：
- 失败步骤：
- 错误信息：
- 可能根因：
- 证据路径：
- 是否可复现：

## 5. 风险评估
- 当前阻塞上线问题：
- 高风险未覆盖区域：
- 可能的回归风险：

## 6. 产物清单
- HTML 报告路径：
- trace 路径：
- video 路径：
- screenshot 路径：
- 其他附件：

## 7. 建议下一步
- 立即修复项：
- 建议补测项：
- 建议补充的测试基础设施：

补充要求：
- 如果测试失败，按失败类型归类：
  - 环境问题
  - 选择器问题
  - 业务逻辑问题
  - 数据问题
  - 时序问题
- 不要使用“应该通过”“看起来没问题”之类措辞。
- 只能用真实执行结果下结论。
- 除了在对话中输出摘要，还必须把完整报告保存到 `<REPORT_OUTPUT_FILE>`。
- 如果只提供了 `<TEST_SCOPE>`，必须先说明是如何识别出最终执行命令的。
```

<a id="prompt-8"></a>
## 8. 根据 Jira ticket 自动生成测试场景和测试用例（自动读取 Jira 版）

```text
Use @jira-automation, @testing-qa, and @test-automator to generate test scenarios and test cases from a Jira ticket.

任务目标：
直接从 Jira ticket 自动读取内容，生成高质量、可执行、可追溯的测试场景和测试用例，用于功能测试、集成测试和端到端测试设计。

输入信息：
- Jira Key: <JIRA_TICKET_KEY> 
- Ticket URL: <JIRA_URL>
- 输出文件路径: <OUTPUT_FILE>

执行要求：
1. 必须优先自动读取 Jira ticket 原文。
2. 自动拉取以下信息：
   - Summary
   - Description
   - Acceptance Criteria
   - Issue Type
   - Priority
   - Components / Labels
   - Assignee / Reporter / Status
   - Linked Issues
   - Linked PR / Commits
   - Attachments / Mockups / API Docs
   - Comments / Clarifications
3. 先提炼需求目标、用户角色、业务对象、关键操作。
4. 从 Description、Acceptance Criteria、Comments 中识别：
   - 主成功路径
   - 必填与格式校验
   - 权限/角色差异
   - 状态流转
   - 异常处理
   - 边界条件
   - 依赖系统或外部接口
5. 对每个 Acceptance Criteria 至少映射 1 个测试场景。
6. 生成的测试内容必须区分：
   - 测试场景
   - 详细测试用例
7. 对每个测试用例标记：
   - 优先级：P0 / P1 / P2
   - 测试类型：功能 / 集成 / E2E / 回归
   - 是否适合自动化：是 / 否 / 部分适合
8. 如果某些字段自动读取失败，要明确列出缺失项，不要自行臆造。
9. 如果 ticket 已关联 PR 或 diff，补充建议的回归测试范围。
10. 最终结果必须保存到本地 `<OUTPUT_FILE>`，格式为 Markdown。

输出格式必须严格如下：

# Jira 测试分析

## 1. Ticket 概览
- Jira Key:
- Summary:
- Issue Type:
- Priority:
- Components:
- 当前状态:
- 主要目标:
- 相关角色:
- 关键业务对象:

## 2. 需求理解
- 需求摘要:
- 核心业务规则:
- 显式 Acceptance Criteria:
- 隐含测试点:
- 关键风险点:

## 3. 测试场景
### 场景 1：<标题>
- 目标：
- 前置条件：
- 测试类型：
- 优先级：
- 覆盖的 Acceptance Criteria：
- 说明：

### 场景 2：<标题>
- 目标：
- 前置条件：
- 测试类型：
- 优先级：
- 覆盖的 Acceptance Criteria：
- 说明：

## 4. 测试用例
### 用例 1：<标题>
- 用例 ID：<可留空或生成建议 ID>
- 关联 Jira：<JIRA_KEY>
- 所属场景：
- 优先级：P0 / P1 / P2
- 测试类型：功能 / 集成 / E2E / 回归
- 是否适合自动化：是 / 否 / 部分适合
- 前置条件：
- 测试数据：
- 执行步骤：
  1.
  2.
  3.
- 预期结果：
  1.
  2.
  3.

### 用例 2：<标题>
- 用例 ID：
- 关联 Jira：
- 所属场景：
- 优先级：
- 测试类型：
- 是否适合自动化：
- 前置条件：
- 测试数据：
- 执行步骤：
  1.
  2.
  3.
- 预期结果：
  1.
  2.
  3.

## 5. 自动化建议
- 建议优先自动化的场景：
- 推荐自动化层级：
  - 单元测试
  - 集成测试
  - E2E 测试
- 不建议自动化的部分：
- 原因：

## 6. 回归测试建议
- 建议回归范围：
- 如果有关联 PR / diff，需要重点回归的模块：
- 风险说明：

## 7. 缺失信息 / 待确认项
- 缺失项：
- 为什么影响测试设计：
- 建议向产品 / 开发确认的问题：

补充要求：
- 不要只输出泛泛的测试点，要输出可以执行的测试场景和测试用例。
- 优先覆盖业务风险高、用户影响大的路径。
- 若 ticket 涉及 UI 表单，必须覆盖：
  - 必填校验
  - 格式校验
  - 成功提交
  - 错误提示
- 若 ticket 涉及状态流转，必须覆盖：
  - 合法流转
  - 非法流转
  - 边界状态
- 若 ticket 涉及权限，必须覆盖：
  - 有权限
  - 无权限
  - 不同角色差异
- 除了在对话中输出摘要外，还必须把完整结果写入 `<OUTPUT_FILE>`。
```

<a id="prompt-9"></a>
## 9. 根据 Jira ticket 自动生成测试场景和测试用例（手工输入版）

```text
Use @testing-qa and @test-automator to generate test scenarios and test cases from the manually provided Jira ticket fields.

任务目标：
在当前环境不能自动访问 Jira，或你不想依赖 Jira 集成时，直接基于手工输入的 Jira ticket 信息生成高质量、可执行、可追溯的测试场景和测试用例。

输入信息：
- Jira Key: <JIRA_KEY>
- Ticket URL: <JIRA_URL>
- Summary: <SUMMARY>
- Description: <DESCRIPTION>
- Acceptance Criteria: <ACCEPTANCE_CRITERIA>
- Issue Type: <ISSUE_TYPE>
- Priority: <PRIORITY>
- Components: <COMPONENTS>
- Labels: <LABELS>
- Assignee: <ASSIGNEE>
- Reporter: <REPORTER>
- Status: <STATUS>
- Linked Issues: <LINKED_ISSUES>
- Linked PR / Commits: <LINKED_PRS_OR_COMMITS>
- Attachments / Mockups / API Docs: <ATTACHMENTS>
- Comments / Clarifications: <COMMENTS>
- 输出文件路径: <OUTPUT_FILE>

执行要求：
1. 不要尝试自动访问 Jira，直接基于我提供的字段工作。
2. 如果某个字段为空或缺失，不要停止任务，要基于现有字段继续分析。
3. 先提炼需求目标、用户角色、业务对象、关键操作。
4. 从 Description、Acceptance Criteria、Comments 中识别：
   - 主成功路径
   - 必填与格式校验
   - 权限/角色差异
   - 状态流转
   - 异常处理
   - 边界条件
   - 依赖系统或外部接口
5. 对每个 Acceptance Criteria 至少映射 1 个测试场景。
6. 生成的测试内容必须区分：
   - 测试场景
   - 详细测试用例
7. 对每个测试用例标记：
   - 优先级：P0 / P1 / P2
   - 测试类型：功能 / 集成 / E2E / 回归
   - 是否适合自动化：是 / 否 / 部分适合
8. 如果 ticket 信息不足，要明确列出缺失项，不要自行臆造。
9. 如果 ticket 已关联 PR 或 diff，补充建议的回归测试范围。
10. 最终结果必须保存到本地 `<OUTPUT_FILE>`，格式为 Markdown。

输出格式必须严格如下：

# Jira 测试分析

## 1. Ticket 概览
- Jira Key:
- Summary:
- Issue Type:
- Priority:
- Components:
- 当前状态:
- 主要目标:
- 相关角色:
- 关键业务对象:

## 2. 需求理解
- 需求摘要:
- 核心业务规则:
- 显式 Acceptance Criteria:
- 隐含测试点:
- 关键风险点:

## 3. 测试场景
### 场景 1：<标题>
- 目标：
- 前置条件：
- 测试类型：
- 优先级：
- 覆盖的 Acceptance Criteria：
- 说明：

### 场景 2：<标题>
- 目标：
- 前置条件：
- 测试类型：
- 优先级：
- 覆盖的 Acceptance Criteria：
- 说明：

## 4. 测试用例
### 用例 1：<标题>
- 用例 ID：<可留空或生成建议 ID>
- 关联 Jira：<JIRA_KEY>
- 所属场景：
- 优先级：P0 / P1 / P2
- 测试类型：功能 / 集成 / E2E / 回归
- 是否适合自动化：是 / 否 / 部分适合
- 前置条件：
- 测试数据：
- 执行步骤：
  1.
  2.
  3.
- 预期结果：
  1.
  2.
  3.

### 用例 2：<标题>
- 用例 ID：
- 关联 Jira：
- 所属场景：
- 优先级：
- 测试类型：
- 是否适合自动化：
- 前置条件：
- 测试数据：
- 执行步骤：
  1.
  2.
  3.
- 预期结果：
  1.
  2.
  3.

## 5. 自动化建议
- 建议优先自动化的场景：
- 推荐自动化层级：
  - 单元测试
  - 集成测试
  - E2E 测试
- 不建议自动化的部分：
- 原因：

## 6. 回归测试建议
- 建议回归范围：
- 如果有关联 PR / diff，需要重点回归的模块：
- 风险说明：

## 7. 缺失信息 / 待确认项
- 缺失项：
- 为什么影响测试设计：
- 建议向产品 / 开发确认的问题：

补充要求：
- 不要只输出泛泛的测试点，要输出可以执行的测试场景和测试用例。
- 优先覆盖业务风险高、用户影响大的路径。
- 若 ticket 涉及 UI 表单，必须覆盖：
  - 必填校验
  - 格式校验
  - 成功提交
  - 错误提示
- 若 ticket 涉及状态流转，必须覆盖：
  - 合法流转
  - 非法流转
  - 边界状态
- 若 ticket 涉及权限，必须覆盖：
  - 有权限
  - 无权限
  - 不同角色差异
- 除了在对话中输出摘要外，还必须把完整结果写入 `<OUTPUT_FILE>`。
```

<a id="recommended-order"></a>
## 推荐使用顺序

1. 如果是全新分析，先用“根据代码生成端到端测试场景”
2. 如果是本地已有 diff，优先用“基于 PR diff 生成端到端测试场景”
3. 如果只有 GitHub PR 链接，优先用“基于 GitHub PR URL 自动拉取 diff 并生成端到端测试场景”
4. 如果希望从 GitHub PR URL 直接落地测试代码，优先用“基于 GitHub PR URL 直接生成 Playwright 测试代码”
5. 如果 Jira 可访问，优先用“根据 Jira ticket 自动生成测试场景和测试用例（自动读取 Jira 版）”
6. 如果 Jira 不可访问，使用“根据 Jira ticket 自动生成测试场景和测试用例（手工输入版）”
7. 再用“根据测试场景生成 Playwright 测试骨架”
8. 再用“根据测试场景生成 Playwright 测试代码”
9. 最后用“运行测试并生成集成测试报告”

<a id="usage-notes"></a>
## 使用建议

- 如果是新项目，先补路由、鉴权、页面、service 这些输入文件，再用第 1 份 prompt。
- 如果是 PR 驱动测试分析，优先使用第 2 份 prompt，并传入明确的 `BASE...HEAD` diff。
- 如果你只有 GitHub PR URL，没有本地 diff，优先使用第 3 份 prompt，并确保本机已登录 `gh auth login`。
- 如果你想从 GitHub PR URL 一步直达 Playwright 代码，使用第 4 份 prompt。
- 如果需求来源是 Jira ticket，且当前环境能访问 Jira，优先使用第 8 份 prompt。
- 如果需求来源是 Jira ticket，但当前环境不能访问 Jira，就把 Jira 字段手工填入第 9 份 prompt。
- 如果是已有 Playwright 项目，第 5 份 prompt 里的“是否已有 Playwright”填 `YES`。
- 如果项目依赖登录态、测试账号、第三方 SSO，先准备好测试环境和账号信息，否则第 4、7、8、9 份 prompt 容易卡在 blocker。
- 第 7 份 prompt 必须建立在真实测试命令和真实输出之上，不能拿来写“假报告”。
- 目前本文档所有 prompt 都已经补充为：结果不仅要在对话中输出，还要明确写入本地目标文件或目标目录。
