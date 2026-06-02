---
name: "uniapp-vue3-wot-design-uni"
description: "规范 UniApp Vue3 项目的页面、平台适配、请求、状态和样式输出。Invoke when 用户在做小程序/App/uni-app，或涉及 pages.json、分包、条件编译、wot-design-uni。"
---

# UniApp Vue3 Wot Design Uni

这是一个“能把模型输出钉死”的 UniApp Skill，不是科普文。

你必须把回答做成跨端工程决策：`pages.json` 改动、平台差异收口、条件编译量化、请求语义、登录态、移动端特有状态、wot 引入方式。任何只讲组件不讲落点的输出都算失败。

## 适用范围

- uni-app Vue3 项目：微信小程序、App、H5。
- `pages.json`、分包、平台适配、请求封装、wot-design-uni 使用。
- 移动端列表/详情/表单/设置页。

## 不适用范围

- 后台 Web 管理端
- 普通 Vue3 Web 项目
- 仅做 UI 点评但不涉及 uni-app 结构

## 触发规则（硬规则）

- 若用户明确提到：uni-app/小程序/App/pages.json/分包/条件编译/wot-design-uni/APP-PLUS，则进入本 Skill。
- 若用户明确提到：后台/管理端/Element Plus/Pinia/权限路由，则拒绝并引导到 `admin-console-vue3-elementplus-ts`。
- 若同时提到：先要求拆为两个独立需求，再分别回答。

## 进入前必问（硬规则）

缺一项就必须先追问，最多追问 2 轮，仍缺则只输出“最小落地骨架 + 风险点”，禁止写大段方案。

必问三项：

- 页面路径或业务域
- 端能力：登录、分享、定位、蓝牙、相机、推送等
- 是否分包，或已有分包规则

若是存量项目，再补问两项：

- 新增页面还是旧页面改造
- 是否已有统一 request 层与平台适配层

## wot 引入方式（必须二选一）

不允许让用户“自行选择”，必须给出推荐路径：

- 推荐：uni_modules 方式。覆盖式升级，easycom 自动接入。
- 仅在以下情况选 npm：项目已统一使用 npm + Vite 显式依赖管理；或团队明确要求统一依赖锁版。

若两者都未明确：默认走 uni_modules。

## 输出顺序（硬规则）

正式回答必须按以下顺序输出，顺序不允许打乱：

1. `pages.json` 改动点
2. 平台差异收口位置
3. 请求封装与登录态
4. 页面状态语义（按页面类型）
5. wot 引入与必要配置
6. 样式 tokens 与密度
7. 最小代码骨架（仅在必要时）
8. 真机验证点与风险

## 平台差异收口（硬规则）

- 平台判断必须集中：`utils/platform.ts`。
- 平台实现必须集中：`utils/adapter/*`，按能力组织（如 `storage.ts`、`share.ts`），不按端组织。
- 业务页面禁止出现 `#ifdef APP-PLUS` / `#ifdef MP-WEIXIN` 等条件编译。
- 仅在 `platform.ts` 与 `adapter/*` 内允许条件编译；若同一能力在 `adapter` 里超过 3 个端实现，必须抽公共逻辑。

## 请求与登录态（硬规则）

必须显式说明：

- 401 / 登录失效后的处理：跳登录页 or 静默续签，二选一并说明推荐策略。
- 跨端 cookie 处理差异：App 用持久化 cookie，微信小程序由 wx.request 自动管理；必须说明本项目选择哪一套。
- 错误语义至少分：网络错误、业务错误、登录失效；每种必须给恢复动作（重试/去登录/降级）。

若用户未答：默认推荐“小程序侧静默续签 + 必要跳登录；App 侧用统一 token + 401 跳登录”，并在风险里说明。

## 状态语义（按页面类型，硬规则）

- 列表页：loading / empty / error / 下拉刷新 / 触底加载；empty 区分无数据与查询无结果；error 必须有重试。
- 详情页：loading / error；empty 区分不存在与无权限。
- 表单页：提交 loading、校验错误、接口错误、离开页面未保存提示（若涉及编辑）。
- 设置页：尽量内联反馈，避免大弹窗。

## 样式与单位（硬规则）

- 设计稿基准：750rpx。
- 字号建议：辅助 24rpx、正文 28rpx、标题 32rpx。
- 间距建议：4 的倍数（16rpx/24rpx/32rpx）优先。
- 颜色与间距必须走 tokens 或 wot 变量层，禁止散写。

## 命名与目录

标准目录结构：

```text
src/
  api/
  components/
    base/
    biz/
  composables/
  pages/
  static/
  styles/
    tokens.scss
    index.scss
  utils/
    request.ts
    platform.ts
    adapter/
  App.vue
  main.ts
  manifest.json
  pages.json
  uni.scss
```

命名硬规则：

- 页面路径：业务域/页面名，或 kebab-case
- 组件：PascalCase
- wot 组件：保持官方 `wd-` 前缀
- composable：useXxx
- 平台判断文件固定：`utils/platform.ts`

## 冲突仲裁规则

- 当“快”与“规范”冲突：优先保证 `pages.json` 落点 + 平台差异收口 + 状态语义；允许牺牲长代码与长教程。
- 当“跨端混提”与“单端稳定输出”冲突：必须拆分需求，禁止混答。
- 当“uni_modules 简单”与“npm 锁版严格”冲突：默认 uni_modules；只有用户明确 npm 时才切。

## 反例（禁止输出的样子）

- 只讲组件名：`用 wd-button、wd-cell 写就完了`。
- 假装跨端一致：`一套代码三端通用`而不提登录态/cookie 差异。
- 把 `#ifdef` 写满页面：业务页面内散写 `APP-PLUS` / `MP-WEIXIN`。
- 错误一句“自行兼容”：对 App/小程序/H5 差异不展开。

## Vue2 存量策略与禁忌

- 只允许增量策略：双工程并存、纯 TS 内核复用。
- 禁止许诺整站无痛升级。
- 禁止把 Vue2 目录习惯写成未来规范。

## 质量检查清单（验收版）

- 是否在前两段就给出 `pages.json` 改动点
- 是否明确 wot 引入方式二选一
- 是否说明平台差异收口位置
- 是否说明 401/登录失效的处理与跨端 cookie 策略
- 是否按页面类型给状态语义与恢复动作
- 是否禁止业务页面散写 `#ifdef` 和 DOM API
- 是否给出真机验证点与风险
