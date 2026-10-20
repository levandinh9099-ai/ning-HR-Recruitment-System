# Ning · 社招招聘工作空间

[English](README.en.md) | 简体中文

[![UAT](https://img.shields.io/badge/UAT-live-2ea44f)](http://47.99.166.93/)
[![Version](https://img.shields.io/badge/version-v0.2.6%2B98cb414-blue)](http://47.99.166.93/api/v1/system/capabilities)
[![Tests](https://img.shields.io/badge/tests-85%20unit%20%7C%2033%20e2e-brightgreen)](docs/verification.md)
[![Stack](https://img.shields.io/badge/stack-React%20%7C%20NestJS%20%7C%20TypeScript-3178c6)](package.json)

一个面向社会招聘的全栈演示系统：从候选人注册、简历维护，到岗位投递、面试排期、Offer、入职和人才库复用，覆盖一条完整可追踪的招聘链路。

- **当前 UAT**：<http://47.99.166.93/>
- **UAT 版本**：`v0.2.6+98cb414`
- **形态**：React SPA + NestJS API + 纯 TypeScript 领域层 + JSON 单文件仓库
- **定位**：可用于演示、评审和 UAT 验证，**不是已完成生产加固的 ATS**

> 当前 UAT 使用 HTTP、演示短信验证码和 JSON 数据卷，仅用于内测，请勿录入真实候选人个人信息。

## 目录

- [项目简介](#项目简介)
- [功能地图](#功能地图)
- [技术栈](#技术栈)
- [系统架构](#系统架构)
- [目录结构](#目录结构)
- [快速开始](#快速开始)
- [演示账号与数据](#演示账号与数据)
- [简历中心](#简历中心)
- [测试与验收](#测试与验收)
- [API 概览](#api-概览)
- [部署与运维](#部署与运维)
- [安全边界与已知限制](#安全边界与已知限制)
- [文档索引](#文档索引)
- [后续路线](#后续路线)
- [许可与说明](#许可与说明)

## 项目简介

「Ning」把招聘流程拆成三个角色共同使用的清晰工作空间：

- **候选人**：注册/登录、维护标准简历、申请岗位、确认面试、查看 Offer 与入职材料；
- **HR**：岗位管理、候选者筛选、面试排期、Offer、入职、人才库与账号权限；
- **业务经理**：查看分管岗位、参与面试评价和候选人推进，但看不到薪酬等敏感字段。

系统强调三件事：

1. **流程有状态**：投递、面试、Offer、入职都有明确的流转、前置条件和阻塞原因；
2. **数据有边界**：候选者、HR、业务经理按角色和岗位裁剪可见数据；
3. **基础设施可替换**：领域逻辑不依赖 HTTP/React，仓库、通知、解析、存储通过端口隔离，便于后续替换为 PostgreSQL、Redis、S3 和真实短信服务。

## 功能地图

### 候选人端

- 手机号验证码登录、新用户自助注册、邀请激活；
- 新简历创建、未绑定档案认领、多档案合并确认；
- **我的简历**：标准结构化简历编辑，右侧 A4 实时预览；
- PDF / DOCX / Markdown / TXT 简历导入、解析核对、字段应用；
- 简历 PDF 下载与浏览器打印；
- 浏览岗位、投递、查看投递进度；
- 面试确认、改期、Offer 查看与接受/拒绝；
- 入职材料提交与状态查看；
- 移动端底部导航、触控尺寸和安全区适配。

### HR / 业务经理端

- 工作概览：在途流程、待办、阶段分布和异常提示；
- 岗位：草稿、发布、暂停、恢复、关闭，负责人和业务经理配置；
- 候选者：按阶段/岗位/待办筛选，查看档案、简历和完整时间轴；
- 投递流转：初筛、淘汰原因、面试推进、Offer、入职；
- 面试：可配置轮数、排期冲突校验、候选人确认/改期、三维评价；
- 面试日历：月/周/日/列表视图，经理按分管岗位裁剪可见范围；
- Offer：岗位/状态/薪酬筛选、版本管理、发送与撤回、接受/拒绝；
- 入职：材料登记、核销/驳回、缺件阻断；
- 人才库：重新发起面试、历史记录只读保留；
- 账号与权限：邀请制账号、`account:manage` 三席委托、会话撤销、密码重置、审计；
- 报表：当前阶段分布与 CSV 导出（本地演示）。

### 关键业务规则

- 同人同岗在途投递拦截；
- 投递记录保存岗位快照，后续岗位修改不影响历史投递；
- 面试时间未到也可在确认后评价并推进；
- 归档候选人隐藏投递并停用登录，可恢复；归档不占用手机号/邮箱唯一性；
- Offer 版本不可覆盖，撤回后生成新版本；
- 入职缺材料返回明确阻塞码，而不是静默失败；
- 账号管理员最多 3 名（含停用账号），至少保留 1 名可用管理员；
- 普通 HR 默认没有账号管理入口和接口权限。

## 技术栈

| 分层 | 技术 |
| --- | --- |
| 前端 | React 18、TypeScript、Vite 7、Ant Design 5、antd-mobile、TanStack Query 5、Zustand 4 |
| 服务端 | NestJS 11、Express、Zod、Multer |
| 领域层 | 纯 TypeScript 用例、权限、状态机、事务编排 |
| 数据 | JSON 单文件仓库（原子替换、串行事务）、Prisma schema 设计稿（未接入运行时） |
| 认证 | Argon2id 密码、短信验证码、访问令牌 + HttpOnly 刷新令牌 |
| 简历 | PDF.js、Mammoth、PDFKit、Noto Sans SC 静态字体 |
| 测试 | Node test runner、Playwright |
| 部署 | Docker Compose、Nginx、命名数据卷、备份脚本 |

版本以 `package.json` 和 `package-lock.json` 为准。

## 系统架构

```text
┌─────────────────────────────────────────────────────┐
│                    React Web App                    │
│  Vite + Ant Design + TanStack Query + Zustand        │
│  候选人 / HR / 业务经理工作台、A4 简历预览            │
└───────────────────────┬─────────────────────────────┘
                        │ /api/v1（同源代理）
┌───────────────────────▼─────────────────────────────┐
│                 NestJS API Controller                │
│  鉴权、参数校验、HTTP 映射、文件上传、PDF 下载        │
└───────────────────────┬─────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────┐
│              Domain Service（纯 TypeScript）          │
│  用例编排 / 权限裁剪 / 状态流转 / 事务 / 审计         │
│  ├─ 招聘流程与投递                                   │
│  ├─ 面试、Offer、入职、人才库                        │
│  ├─ 注册、邀请、账号权限                             │
│  └─ 标准简历与简历导入                               │
└───────────────┬──────────────────────┬──────────────┘
                │                      │
     ┌──────────▼─────────┐   ┌────────▼──────────┐
     │ JsonRepository     │   │ Local services    │
     │ .data/*.json       │   │ 临时文件、PDF、审计 │
     └────────────────────┘   └───────────────────┘
```

领域层不依赖 React 或 NestJS，测试可以直接调用用例；基础设施适配器通过 `packages/domain/ports.ts` 的接口接入。

## 目录结构

```text
apps/
  api/                    NestJS 入口、认证、会话、仓库、简历导入、PDF
    assets/               Noto Sans SC Regular/Bold 字体
  web/                    React 应用、页面、API 客户端、预览组件
    pages/                登录、概览、岗位、候选者、简历、面试、Offer、人才库等
packages/domain/
  model.ts                领域实体、状态、错误
  service.ts              用例、权限、事务编排
  transitions.ts          状态流转和前置条件
  consents.ts             协议版本与同意项
  phone.ts                手机号 HMAC
  ports.ts                仓库/通知/存储/评分等端口
  seed.ts                 相对当前时间生成的演示数据
infra/
  schema.prisma           生产 PostgreSQL 模型设计稿
  constraints.sql          补充约束设计
  compose.yaml            PostgreSQL/Redis/MinIO 开发依赖
tests/
  *.test.ts               领域与 API 单元测试
  e2e/                    Playwright 浏览器回归
deploy/
  Dockerfile.api          API 镜像
  Dockerfile.web          Web 构建 + Nginx
  compose.yaml            UAT 编排
  env.example             环境变量模板
  backup.sh               数据卷备份
docs/                     产品、技术、实现记录与验证文档
templates/                标准简历模板文件
```

## 快速开始

### 环境要求

- Node.js **22.12+**（开发和验证使用 Node.js 24）
- npm
- 可选：Docker（部署或运行基础设施依赖）

### 本地开发

```bash
npm ci
npm run dev
```

启动后：

- Web：<http://127.0.0.1:5173>
- API：<http://127.0.0.1:3001/api/v1/health>

前端通过 Vite 代理访问同源 `/api`。本地服务默认只绑定回环地址。

### 常用命令

```bash
npm run dev          # 同时启动 API 和 Vite
npm run dev:api      # 仅 API
npm run dev:web      # 仅 Web
npm test             # 领域与 API 单元测试
npm run build        # TypeScript 严格检查 + 前端生产构建
npm run test:e2e     # Playwright 浏览器回归
```

首次运行 E2E 前安装浏览器：

```bash
npx playwright install chromium
```

E2E 会自动启动独立的 API（3002）和前端（5174），使用独立的临时 JSON 数据文件，不会污染本地 `.data/recruitment.json`。

## 演示账号与数据

### 内部账号（本地默认）

| 角色 | 账号 | 本地演示密码 |
| --- | --- | --- |
| HR 管理员 | `hr.admin` | `Recruit@2026!` |
| 普通 HR | `hr.user` | `Recruit@2026!` |
| 业务经理 | `manager1` | `Recruit@2026!` |

UAT 账号和密码由部署环境单独配置，不写在本仓库中。

### 候选人演示手机号

本地演示候选人为 `13800000001` 至 `13800000012`，对应：

林知夏、陈一鸣、周予安、许清禾、沈亦舟、苏晚晴、程景行、陆星遥、顾言、唐可欣、孟知远、叶青。

在候选者登录页点击“获取验证码”，页面会显示本地演示验证码；也可用任意新手机号自助注册。

演示黑名单手机号为 `13800009999`。频率限制、图形验证码、黑名单为单进程内存实现，重启后清零。

### 数据文件

- 本地：`.data/recruitment.json`
- UAT：Docker 命名卷 `recruit-uat_recruit_data`
- E2E：`.data/e2e-*.json`

数据文件已被 Git 忽略。停掉服务后重命名 `.data/recruitment.json`，下次启动会重新生成种子数据。

## 简历中心

简历模块是当前系统的核心能力之一。

### 编辑与预览

- 标准结构化简历：基本信息、求职意向、个人简介、核心能力、工作经历、项目经历、教育背景、在校经历、荣誉奖项、技能证书、自我评价、兴趣爱好；
- 姓名、电话、邮箱、学历、工作年限已合并到“我的简历”页面，保存时同步候选人档案；
- 左右两栏：左侧编辑区独立滚动，右侧 A4 预览独立滚动；
- 预览使用与 PDF 相同的版式：黑色标题、深绿色小方块、日期在左/组织在右、标准黑色正文；
- 右侧预览按内容实时更新，空栏目自动隐藏。

### 简历导入

支持：

- `.pdf`（文本型，扫描件不支持）
- `.docx`
- `.md` / `.txt`

流程：

1. 上传文件（最大 10 MB）；
2. 服务端使用 PDF.js / Mammoth 提取文本；
3. 规则解析目标岗位、基本信息、工作、项目、教育、技能等字段；
4. 在核对弹窗中查看字段、置信度和解析预览；
5. 选择应用方式：
   - 合并；
   - 替换选中字段；
   - 新建版本；
6. 写入标准简历，进入正常编辑和导出。

解析结果存储在 `.data/resume-imports/` 或 `RESUME_IMPORT_DIR`，默认保留 24 小时。

### PDF 导出

- 服务端使用 PDFKit 生成真实 `application/pdf`；
- 使用 Noto Sans SC 静态 Regular/Bold 字体，避免可变字体默认 Thin 字重导致文字过淡；
- PDF 章节顺序、字段范围、日期/组织排版与右侧预览对齐；
- 支持直接下载 PDF 与浏览器打印另存为 PDF；
- 测试会提取 PDF 文本，逐项校验所有章节和代表内容。

### 已知限制

- 规则解析，不包含 OCR 或外部大模型；
- 复杂双栏、表格、图片型简历可能识别不完整；
- 扫描件上传会返回“暂不支持扫描件”；
- 附件仅保存本地临时文件，不进入对象存储；
- 暂不支持候选人附件在线预览。

## 测试与验收

### 测试结果

| 命令 | 结果 |
| --- | --- |
| `npm test` | **85 / 85 通过** |
| `npm run test:e2e` | **33 / 33 通过** |
| `npm run build` | 通过 |
| `prisma validate` | 通过（仅 schema 校验） |

覆盖范围包括：

- 招聘闭环、状态守卫、归档/恢复、投递快照；
- 注册、邀请、协议、黑名单、图形验证码、账号状态；
- 岗位发布条件、动作能力阻塞、可配置面试轮数；
- 日历时间边界、跨日、权限、日桶、取消；
- 候选人排序、人才库重新发起、旧结果不变；
- Offer 查询、薪酬/版本/权限；
- `account:manage` 三席委托、账号查询、会话撤销、密码重置；
- 标准简历保存、版本冲突、中文 PDF 生成；
- DOCX/PDF 简历解析、基本信息解析与字段应用；
- 人才库查看候选者档案；
- Ning 品牌资源、登录页移动端展示与品牌展示板；
- 移动端底部导航、390px 布局、日历日期条、Offer 筛选；
- 浏览器回归使用独立临时数据，避免相互污染。

详细证据见 [测试与验收记录](docs/verification.md) 和 [UAT 部署记录](docs/UAT部署记录_2026-10-18_v0.2.1.md)。

## API 概览

API 基址：`/api/v1`。认证使用 `Authorization: Bearer <accessToken>`，刷新使用 HttpOnly Cookie。

| 领域 | 代表接口 |
| --- | --- |
| 健康/能力 | `GET /health`、`GET /system/capabilities` |
| 认证 | `POST /auth/login`、`POST /auth/sms/send`、`POST /auth/sms/login`、`POST /auth/refresh`、`POST /auth/logout` |
| 注册/邀请 | `GET /auth/invite/:token`、`POST /auth/invite/accept`、`POST /auth/password-reset/accept` |
| 工作空间 | `GET /workspace`、`GET /directory/staff` |
| 岗位 | `POST /jobs`、`PATCH /jobs/:id` |
| 投递 | `POST /applications`、`POST /applications/:id/transitions`、`GET /applications/:id/actions` |
| 面试 | `GET /interviews/calendar`、`GET /interviews/:id` |
| 简历 | `GET/PUT /candidates/me/resume`、`GET /candidates/me/resume.pdf`、`GET /candidates/:id/resume` |
| 简历导入 | `POST /candidates/me/resume/imports`、`GET /candidates/me/resume/imports/:id`、`POST .../apply`、`POST .../retry` |
| Offer | `POST /offers/query`、`GET /offers/:id`、`GET /offers/filter-options` |
| 人才库 | `GET /talent`、`POST /talent/:entryId/reactivation-preview`、`POST /talent/:entryId/reactivations` |
| 账号管理 | `GET/POST /admin/accounts`、`POST /admin/accounts/query`、`POST /admin/accounts/:id/password-reset`、`POST /admin/accounts/:id/sessions/revoke` |

接口参数和权限以领域服务和测试为准。

## 部署与运维

### UAT

- 地址：<http://47.99.166.93/>
- 版本：`v0.2.4+4fc66bf`
- 形态：`api` + `web` 两个 Docker 服务，Nginx 监听 80，API 内网 3001；
- 数据卷：`recruit-uat_recruit_data`；
- 健康检查：<http://47.99.166.93/api/v1/health>

### Docker Compose

```bash
cd deploy
cp env.example .env
# 编辑 APP_BASE_URL、PHONE_HMAC_SECRET、SEED_PASSWORD、DEMO_MODE、COOKIE_SECURE
docker compose up -d --build
docker compose ps
curl -fsS http://127.0.0.1/api/v1/health
```

### 环境变量

| 变量 | 说明 |
| --- | --- |
| `NODE_ENV` | `production` 关闭开发行为 |
| `API_HOST` / `API_PORT` | API 监听地址和端口 |
| `RECRUIT_DATA_PATH` | JSON 数据文件路径，容器内默认 `/data/recruitment.json` |
| `APP_BASE_URL` | 邀请链接和通知中的站点地址 |
| `PHONE_HMAC_SECRET` | 手机号黑名单 HMAC 密钥，上线后不要更换 |
| `SEED_PASSWORD` | 首次生成固定内部账号的初始密码 |
| `DEMO_MODE` | `1` 返回演示验证码并补齐演示候选账号 |
| `COOKIE_SECURE` | HTTPS 后设为 `1`；HTTP 内测为 `0` |
| `TRUST_PROXY` | 信任 Nginx 的 `X-Forwarded-For` |
| `RESUME_IMPORT_DIR` | 简历上传临时目录，默认为 `.data/resume-imports` |
| `RESUME_FONT_PATH` / `RESUME_BOLD_FONT_PATH` | 可选：覆盖 PDF 正文字体和粗体路径 |
| `APP_ENV` / `BUILD_VERSION` / `DEPLOYED_AT` | 系统能力页和页脚展示的环境信息 |

### 备份

```bash
# 服务器上执行；脚本保留最近 7 天
BACKUP_DIR=/root/recruit-backups /opt/recruit/deploy/backup.sh

# 每日 03:00
( crontab -l 2>/dev/null; echo '0 3 * * * /opt/recruit/deploy/backup.sh >> /var/log/recruit-backup.log 2>&1' ) | crontab -
```

备份文件包含手机号等个人信息，必须加密保管。部署前建议同时备份环境变量和数据卷。

### 域名与 HTTPS

`ljntt.top` 已完成 DNS 配置，但当前服务器仍受阿里云 ICP 备案拦截，UAT 继续使用 IP。备案和 HTTPS 完成前不要对外宣传域名；HTTPS 配置后需把 `COOKIE_SECURE` 改为 `1`。

## 安全边界与已知限制

这是一个**演示/UAT 系统**，不应直接作为生产 ATS 上线：

- JSON 单文件仓库，单进程串行事务，不支持多实例、水平扩展或生产级并发；
- 访问令牌保存在 API 进程内存，重启后失效；刷新令牌摘要和审计在 JSON 中；
- 演示短信验证码通过接口返回，没有接入真实短信服务；
- 没有网关限流、CSRF 防护、不可变审计和密钥托管；
- 数据文件未加密，备份包含明文个人信息；
- 简历解析为规则解析，没有 OCR/外部 LLM；
- 附件、邮件、通知仍为本地/站内演示能力；
- 生产数据库模型、Prisma Repository、Redis/BullMQ、S3 预签名上传尚未接入；
- 报表为当前阶段分布，不是完整历史转化漏斗。

生产化至少需要补齐：PostgreSQL/Prisma 事务、Redis/JWT、真实短信与邮件、对象存储、HTTPS/WAF、审计、MFA/SSO、备份恢复演练和隐私合规。

## 文档索引

| 文档 | 内容 |
| --- | --- |
| [架构说明](docs/architecture.md) | 需求取舍、数据决策、扩展方式和上线边界 |
| [Ning 品牌设计](docs/Ning品牌设计.md) | 标识、字标、配色、使用规范与接入位置 |
| [注册系统](docs/registration.md) | 注册、邀请、协议、风控和部署准备 |
| [v0.2 迭代记录](docs/iteration-v0.2.md) | A 批次实现范围和待办 |
| [测试与验收](docs/verification.md) | 测试命令、覆盖范围和验收证据 |
| [UAT 部署记录](docs/UAT部署记录_2026-10-18_v0.2.1.md) | 版本、备份、冒烟验证和已知边界 |
| [简历自动解析产品设计](docs/简历自动解析_产品设计文档_v0.1.md) | 解析范围、字段、交互和验收 |
| [简历自动解析技术栈](docs/简历自动解析_技术栈文档_v0.1.md) | 解析管线、存储、错误和测试设计 |
| [简历自动解析实现记录](docs/简历自动解析_实现记录_v0.1.md) | 已实现字段、测试和限制 |
| [简历标准模板实现记录](docs/简历标准模板_系统化实现记录_v0.1.md) | 结构化简历、预览和 PDF 版式 |
| [账号与权限 P0](docs/账号与权限_P0实现记录_v0.1.md) | 账号查询、会话、密码重置和权限委托 |
| [账号权限重设计](docs/账号与权限重设计方案_v0.2.md) | 三角色权限模型的后续设计 |
| [Offer 管理筛选](docs/Offer管理筛选_实现记录_v0.1.md) | 岗位/状态/薪酬筛选和权限 |
| [面试日历](docs/面试日历_实现记录_v0.1.md) | 月/周/日/列表视图、排期和权限 |
| [人才库重新面试](docs/人才库重新面试_实现记录_v0.1.md) | 重新发起、历史保护和验收 |
| [部署说明](deploy/README.md) | Docker、Nginx、数据卷、备份和发版 |

## 后续路线

### P0：可生产运行

- PostgreSQL + Prisma Repository，替换 JSON 仓库；
- Redis 会话/验证码/频控 + JWT 密钥托管；
- 真实短信、邮件、对象存储与附件预览；
- HTTPS、WAF、CSRF、限流、不可变审计；
- 完整备份恢复和隐私合规。

### P1：招聘效率

- 真实 OCR / 大模型简历解析与字段置信度校准；
- 异步任务队列、批量 CSV 邀请、定时催办；
- 面试评价模板、Offer 审批流、入职附件；
- MFA、SSO、访问权限定期复核。

### P2：数据与体验

- 招聘漏斗、渠道转化、周期分析；
- 简历模板主题和品牌化；
- 更多移动端快捷操作和通知中心。

## 许可与说明

本项目为招聘产品演示与课程/项目交付用途，仓库未声明开源许可证。如需复用代码、数据模型或界面，请先联系项目维护者确认授权范围。

演示数据中的姓名、手机号、邮箱、简历均为虚构数据，请勿替换为真实个人信息。
