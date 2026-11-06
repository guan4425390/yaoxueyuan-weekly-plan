# 药学院每周工作计划

手机端优先的工作计划填报与汇总系统。老师按姓名登录，在日历上点任意一天填写当天计划；管理员可查看全部记录、管理账号、导出汇报级 HTML 报告。

**正式地址：<https://yaoxueyuan-weekly-plan.pages.dev/>**

> 目前完全跑在 Cloudflare（Pages + D1），**不依赖 WorkBuddy 后端，也没有任何第三方令牌**。
> GitHub 仓库 <https://github.com/guan4425390/yaoxueyuan-weekly-plan> 只作为代码存档；打开 <https://guan4425390.github.io/yaoxueyuan-weekly-plan/> 会自动跳到上面的正式地址。

## 功能

| 功能 | 说明 |
| --- | --- |
| 登录 | 账号为教师姓名，直接手动输入；初始密码 `666666`，首次登录建议自行修改 |
| 保持登录 | 登录时勾选「保持登录状态」，本机记住设备号，下次打开免密码（令牌 180 天，换设备自动失效） |
| 日历 | 覆盖未来五年，按月 / 按年翻页、回到今天；**每一格都可点击**新建计划 |
| 计划录入 | 时间、工作地点（预设下拉 + 自定义）、参与人员、工作事项、备注 |
| 日历标记 | 有计划的日期显示 ①②③ 简略序号，超出显示「+N」，点开看当日明细 |
| 条数上限 | 默认每天 10 条，根管理员可在后台调整 |
| 汇总导出 | 仅管理员可用；按时间分组 / 按人物分组；自定义起止日期或「本周」等快捷；写明星期几 |
| 导出格式 | 生成一份**独立美观的 HTML 报告**，可直接打开、打印或发邮件用于汇报 |
| 账号管理 | 根管理员可新增账号、改角色、重置密码、**解锁被锁账号**、修改或删除其他管理员 |
| 操作记录 | 根管理员可查看所有人「谁 / 什么时候 / 新增或修改了什么」 |
| 存储容量 | 主页日历下方常驻容量徽标（绿 / 黄 / 红），超阈值顶部弹提醒条并可一键去清理 |

## 角色

| 角色 | 权限 |
| --- | --- |
| 根管理员 | 全部权限：导出、账号管理、解锁、操作记录、容量与清空 |
| 管理员 | 导出汇总表、查看全部计划 |
| 教师 / 辅导员 | 填报与查看本人及全院计划 |

连续输错密码会锁定账号，由根管理员在「设置 → 账号」里点「解锁」恢复。

## 架构（Cloudflare Pages + D1）

```
浏览器（单文件 index.html）
    │  fetch /api/*
    ▼
Cloudflare Pages Functions（cloudflare/worker.js）
    │
    ▼
Cloudflare D1（SQLite）── kv 表：k='db' 一行存放全部数据的 JSON
```

| 组成 | 位置 | 说明 |
| --- | --- | --- |
| 前端 | `app.template.html` → `dist/index.html` | 单文件，图片 base64 内嵌 |
| 后端 | `cloudflare/core.js` | 与运行时无关：路由、登录、令牌签发、乐观锁 |
| 入口 | `cloudflare/worker.js` | Pages / Worker 通用；有 D1 用 D1，没有才回落 GitHub |
| 数据 | D1 `yaoxueyuan-plan-db` | 一张 `kv` 表，`rev` 做乐观锁，并发保存不互相覆盖 |
| 登录 | HMAC-SHA256 无状态令牌 | 密钥 `APP_SECRET`；令牌内含设备号，换设备即失效 |

数据文档结构：`plans`（计划）、`accounts`（账号，密码为 `sha256(name|pwd)`，不下发前端）、`settings`（参数）、`logs`（操作记录，最多 300 条）。

## 常用命令

```bash
python tools/build.py            # 生成 index.html + dist，并跑 24 项静态校验
python tools/prep_pages.py       # 生成 Cloudflare 部署目录 .pages-out
npx wrangler pages deploy        # 部署（读 wrangler.toml）
python tools/verify_cf.py https://yaoxueyuan-weekly-plan.pages.dev   # 线上 16 项验证
python tools/init_d1.py          # 把 .data/db.clean.json 生成建表 SQL
python tools/d1_put.py <json>    # 整份数据写回 D1（带版本号）
node tools/dev_api.mjs           # 本地起服务（127.0.0.1:8787），不碰线上
```

数据备份：管理页可导出 HTML / 表格；整份 JSON 也能取（`npx wrangler d1 execute yaoxueyuan-plan-db --remote --json --command "SELECT v FROM kv WHERE k='db'"`）。

---

最后更新：2026-11-07
