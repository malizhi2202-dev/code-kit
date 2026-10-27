> 分片自 4-dev.md · 何时整读我：任务涉及新增表/字段/改类型/索引/外键/重命名/删表删列（R4.5）时，进入实现前整读并按其执行；不读禁止动 schema · 属 code-kit 小文件按需加载体系（RULES.md R1.10）

### 1.7 数据库 Schema 任务额外检查（涉及表 / 字段变更必跑 · 对应 R4.5）

> 这是 AI 开发最高频的事故源——改了 ORM model 没生迁移，跑起来报"表/字段不存在"。本段强制堵住。

**判定标准**：任务的 `action` 含「**新增表 / 加字段 / 改字段类型 / 加索引 / 加外键 / 重命名表/列 / 删表/列**」等关键词，或 `files` 涉及：

- ORM model：`models/*.py` / `*Model.ts` / `entity/*.java` / `*.entity.ts` / `schema.prisma` / `*.gorm.go`
- 迁移目录：`migrations/*` / `db/migrate/*` / `alembic/versions/*` / `prisma/migrations/*`
- DDL：`*.sql`（含 `CREATE TABLE` / `ALTER TABLE` 等）

命中时，进入实现**之前**必须：

#### 1.7.1 声明 schema diff

在执行计划里**显式列出**：

```
## Schema Diff（本任务）
- 新增表：`<table>`（字段：col1 TYPE NOT NULL, col2 TYPE DEFAULT ...）+ 索引：...
- 改字段：`<table>.<col>` 类型 `<old>` → `<new>`，迁移策略：<是否需要 backfill / 兼容期>
- 删字段：`<table>.<col>`，迁移策略：<是否需要先双写过渡>
- 新增外键：`<table>.<col>` → `<ref-table>.<ref-col>`（ON DELETE: ...）
```

**禁止**没声明就开干。

#### 1.7.2 选执行机制（按优先级探测项目）

按下面顺序探测，**用第一个命中的**：

| 优先级 | 检测信号 | 用什么 | 命令示例 |
|---|---|---|---|
| 1 | `prisma/schema.prisma` 存在 | Prisma | `npx prisma migrate dev --name <change-id>_<task-id>` |
| 2 | `alembic.ini` 存在 | Alembic | `alembic revision --autogenerate -m "<change-id> <task-id>"` |
| 3 | `db/migrate/` 目录（Rails） | Active Record | `rails generate migration <CamelName>` |
| 4 | `knexfile.*` 存在 | Knex | `npx knex migrate:make <change-id>_<task-id>` |
| 5 | `flyway.conf` / `flyway/sql/` 存在 | Flyway | 手写 `V<timestamp>__<change-id>_<task-id>.sql` 放进 `flyway/sql/` |
| 6 | `liquibase/changelog.xml` 存在 | Liquibase | 追加 `<changeSet>` 到 changelog |
| 7 | `migrations/` 或 `db/migrations/` 目录（无框架）| 手写 SQL | 文件名：`YYYYMMDDHHmm_<change-id>_<task-id>_<verb>.sql` |
| 8 | 全部都没有（裸项目）| **回退**：生成在 `.specs/<change-id>/migrations/` | 文件名同上，并在 SUMMARY 里登记"待用户搬到正式迁移目录" |

#### 1.7.3 生成可逆迁移

每个迁移文件必须含 `up` 和 `down`（或对应框架的 migrate / rollback）：

```sql
-- up
ALTER TABLE users ADD COLUMN avatar_url VARCHAR(255) DEFAULT NULL;
CREATE INDEX idx_users_avatar_url ON users(avatar_url);

-- down
DROP INDEX idx_users_avatar_url ON users;
ALTER TABLE users DROP COLUMN avatar_url;
```

**禁止** down 段写 `-- 不可回滚`。即使破坏性变更（删字段），down 也要写"重建空字段 + 数据丢失警告"。

#### 1.7.4 检测 DB 凭据 → 决定要不要现在执行

按下面顺序 grep 项目（用 `grep_search` 工具或等价命令）：

```
.env / .env.local / .env.development / .env.example
config/database.yml / config/database.json
application.yml / application.properties / application-*.yml
docker-compose.yml（DB 服务段：postgres / mysql / mariadb / mongo）
prisma/schema.prisma 的 datasource 段
serverless.yml / wrangler.toml（如适用）
```

##### 路径 A · 找到凭据

**必须停下来反问用户**，模板：

```
🔍 检测到本项目有 DB 连接配置（具体：.env 第 12 行 DATABASE_URL）。
本任务包含 schema 变更：
  - 新增表：notifications
  - 改字段：users.email_verified（NULL → NOT NULL DEFAULT FALSE）

是否现在执行迁移？
  1. 现在执行（我会跑：npx prisma migrate dev --name add-notifications）
  2. 只生成 SQL/迁移文件，由你手动跑（推荐生产敏感项目）
  3. 让我先看 SQL 再决定（我会贴出迁移文件内容给你看）
```

- **用户选 1**：AI 跑迁移命令，**贴出真实输出**到 SUMMARY，并跑 `\d <table>` / `DESCRIBE <table>` / `SHOW CREATE TABLE` 验证表结构落地
- **用户选 2 / 3**：AI 生成迁移文件，等用户选定后再决定执行

##### 路径 B · 未找到凭据

直接走"只生成迁移文件"：

1. 按 1.7.2 选定的机制生成迁移
2. 在 `<task-id>-SUMMARY.md`「数据库迁移」段**显式提醒**：

```
⚠️ 数据库迁移文件已生成（未检测到 DB 凭据，需手动执行）
  - 文件：prisma/migrations/20260501_add_notifications/migration.sql
  - 执行命令：npx prisma migrate deploy
  - 验证命令：psql -c "\d notifications"
  - 环境清单（执行后逐项打勾）：
    - [ ] local
    - [ ] dev
    - [ ] staging
    - [ ] prod（生产请走 maintenance window，先 dry-run）
```

#### 1.7.5 反幻觉（R6.1）

**禁止凭印象写迁移**：

- 加字段 → 必须先 grep ORM model 确认字段已声明（避免迁移和 model 不一致）
- 改字段类型 → 必须确认旧类型 → 新类型的兼容性（VARCHAR(50) → VARCHAR(20) 可能截断；BIGINT → INT 可能溢出）
- 加 NOT NULL → 必须确认有 DEFAULT 或先做 backfill（不然旧行报错）
- 加外键 → 必须确认引用列有索引

#### 1.7.6 5-test 关联

本步生成的迁移文件，5-test `@code-kit/prompts/5-test-rounds.md` 第 4 轮「4.2 数据迁移测试」会再验证一次（在生产数据快照上预演）。所以这里生成的文件必须能被 trace（路径要写进 SUMMARY）。
