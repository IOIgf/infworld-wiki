# 🌌 无限世界 · infworld Wiki

欢迎来到 **无限世界** 的官方维基。

这是一个基于 **Flask + SQLAlchemy + PostgreSQL** 开发的**多人角色扮演互动游戏平台**。玩家可以进入由 AI（DeepSeek）驱动的副本，通过文字冒险推进剧情，获取积分、道具，并沿着独特的**树形命途系统**成长。

---

## 📖 目录

1. [项目简介](#项目简介)
2. [核心功能](#核心功能)
3. [技术架构](#技术架构)
4. [数据库设计](#数据库设计)
5. [命途系统（树形）](#命途系统树形)
6. [副本与AI交互](#副本与ai交互)
7. [管理员后台](#管理员后台)
8. [部署与运维](#部署与运维)
9. [开发与扩展](#开发与扩展)
10. [许可证](#许可证)

---

## 项目简介

**无限世界** 是一个让玩家通过文字与 AI 互动，在随机生成的副本中冒险的 Web 应用。它融合了传统的 MUD 风格、AI 叙事和现代 Web 技术。

- **🎯 核心玩法**：玩家进入副本，与 AI 对话，做出选择，使用道具，战斗，最终通关获得积分和命途经验。
- **🌀 命途系统**：每个玩家通过 25 道测试题绑定一个“命途”（如繁荣、死亡、欺诈），命途以**树形结构**展开，玩家可在达到条件时选择进化分支。
- **👥 多人协作**：支持团队副本，玩家可匹配组队，共享剧情，协同作战。
- **💰 经济系统**：积分、商店、交易市场、背包，构成完整的玩家经济循环。
- **📊 管理后台**：完善的权限体系（RBAC），支持管理物品、副本、用户、命途、历史记录等。

---

## 核心功能

| 功能模块       | 说明                                                    |
| -------------- | ------------------------------------------------------- |
| **用户系统**   | 注册/登录、个人主页（Markdown简介）、头像上传、密码修改 |
| **命途系统**   | 25题测试绑定命途，树形分支进化，能力解锁，红点提醒      |
| **副本系统**   | 单人/团队副本，AI 驱动叙事，道具使用，战斗，通关结算    |
| **商店系统**   | 购买道具，道具分组展示，AI 自动生成描述                 |
| **交易市场**   | 挂单出售，购买，5% 手续费                               |
| **聊天系统**   | 世界频道、私信、团队频道                                |
| **排行榜**     | 积分榜、命途经验榜，支持头像展示                        |
| **管理员后台** | 物品/分组/副本/用户/命途/历史/权限/日志管理             |

---

## 技术架构

### 后端

- **Web 框架**: Flask 3.x
- **ORM**: SQLAlchemy 2.x
- **数据库**: PostgreSQL 14+（生产环境），开发可用 SQLite
- **AI 引擎**: DeepSeek API（v4-flash）
- **任务调度**: 内置定时清理（每天一次）
- **部署**: Gunicorn + PM2（可选）

### 前端

- **语言**: 原生 JavaScript（ES6+）
- **样式**: 纯 CSS（双主题：亮/暗）
- **渲染**: 服务端 Jinja2 模板
- **交互**: Fetch API + Server-Sent Events（流式 AI 回复）

### 部署环境

- **OS**: Ubuntu 22.04 LTS
- **Web Server**: Gunicorn（反向代理可选 Nginx）
- **进程管理**: PM2 或 systemd
- **存储**: 文件系统（头像、静态资源）

---

## 数据库设计

### 核心表（共 31 张）

| 表名                   | 说明                     | 关键字段                                                     |
| ---------------------- | ------------------------ | ------------------------------------------------------------ |
| `user`                 | 用户账号、积分、命途绑定 | `id`, `username`, `coins`, `fate_id`, `current_fate_node_id` |
| `fate`                 | 命途定义（根）           | `id`, `name`, `motto`, `color`, `icon`                       |
| `fate_node`            | 命途树形节点             | `id`, `fate_id`, `parent_id`, `name`, `required_level`, `cost_coins` |
| `user_fate_node`       | 玩家解锁的节点           | `user_id`, `node_id`                                         |
| `fate_question`        | 命途测试题（25题）       | `id`, `question_text`, `option_*`, `weight_*`                |
| `user_fate_answer`     | 玩家答题进度             | `user_id`, `answers`(JSON), `current_question`, `submitted`  |
| `dungeon`              | 副本定义                 | `id`, `name`, `rules`, `is_hidden`, `is_team`, `max_team_size` |
| `dungeon_session`      | 单人副本会话             | `user_id`, `dungeon_id`, `history`(JSON)                     |
| `team`                 | 团队                     | `id`, `dungeon_id`, `member_ids`(JSON)                       |
| `team_session`         | 团队副本会话             | `team_id`, `player_id`, `history`(JSON)                      |
| `team_action`          | 团队公开行动记录         | `team_id`, `player_id`, `description`, `is_private`          |
| `team_chat`            | 团队聊天消息             | `team_id`, `sender_id`, `content`                            |
| `item`                 | 道具定义                 | `id`, `name`, `price`, `effect`, `description`, `group_id`   |
| `item_group`           | 道具分组                 | `id`, `name`                                                 |
| `user_item`            | 玩家背包                 | `user_id`, `item_id`, `quantity`                             |
| `order`                | 交易市场订单             | `id`, `seller_id`, `item_id`, `quantity`, `price_per_unit`, `status` |
| `chat_message`         | 世界聊天                 | `id`, `user_id`, `content`, `timestamp`                      |
| `private_message`      | 私信                     | `sender_id`, `receiver_id`, `content`, `timestamp`           |
| `matching_queue`       | 团队匹配队列             | `user_id`, `dungeon_id`, `status`                            |
| `dungeon_clear_record` | 通关记录（首次奖励）     | `user_id`, `dungeon_id`                                      |
| `dungeon_pool_item`    | 副本自定义奖池           | `dungeon_id`, `item_id`, `quantity_min`, `quantity_max`, `drop_rate` |
| `session_history_log`  | 副本历史存档             | `session_type`, `session_id`, `player_id`, `history`(JSON), `cleared` |
| `admin_log`            | 管理员操作日志           | `admin_id`, `action_type`, `target_type`, `target_id`, `details` |
| `permission`           | 权限定义                 | `id`, `key`, `name`                                          |
| `user_permission`      | 用户权限分配             | `user_id`, `permission_id`                                   |
| ……                     | 其他辅助表               | 详见代码                                                     |

---

## 命途系统（树形）

### 设计理念

每个命途是一个树形结构，根节点为初始能力，后续节点为分支进化。玩家从根节点开始，达到条件后选择解锁子节点，形成**个性化路径**。

### 数据结构

- `FateNode`：节点（id, fate_id, parent_id, name, required_level, cost_coins, …）
- `UserFateNode`：玩家解锁记录
- `User.current_fate_node_id`：玩家当前所在节点

### 玩家流程

1. **觉醒**：完成25题 → 绑定命途 → 自动解锁根节点。
2. **查看路径**：在 `/fate/detail` 查看已走过的线性路径（从根到当前节点）。
3. **红点提醒**：当存在可解锁子节点且玩家等级/积分达标时，显示红点。
4. **进化选择**：点击“进化”进入分支选择界面，展示所有当前节点的子节点，达标节点高亮可点击。
5. **解锁**：消耗积分，解锁新节点，更新当前指针，红点消失（若还有下一层则继续出现）。

### 管理员操作

- 在 `/admin/fate` 管理任意命途的树形结构。
- 支持**添加根节点/子节点**（最多5层）、**编辑**、**删除**（自动提升子节点，重置受影响的玩家）。

---

## 副本与AI交互

### 单人副本

- 玩家进入副本 → 创建/恢复 `DungeonSession`。
- 每次发送消息，后端构建 `system_prompt`（含副本规则、命途能力、奖池列表），调用 DeepSeek API。
- AI 回复以 **流式（SSE）** 返回，前端实时显示。
- **道具使用**：前端点击“使用” → 后端扣除数量 → 消息中标注 `[道具使用]` → AI 描述效果，并输出 `consume: 道具名, 数量` 指令（后端解析扣减）。
- **通关判定**：AI 回复包含 `[通关]`、`[积分:数字]`、可选 `[道具:名称]` → 后端解析，发放奖励（积分、道具、命途经验）。
- **首次通关**额外获得 30 命途经验，重复通关仅积分。

### 团队副本

- 匹配：玩家点击匹配 → 进入队列 → 满员后自动组队。
- 会话：每个队员独立 `TeamSession`，但共享 `TeamAction`（公开行动）和 `TeamChat`。
- AI 提示词包含队友列表和近期公开行动，从**当前玩家视角**叙事。
- 通关奖励：**每个队员独立获得积分**（各自命途加成），各自获得首次通关经验（40点）。
- 道具奖励：只发放给当前玩家。

### 战斗系统（内置在 AI 提示词中）

- 玩家属性由命途等级动态计算（生命/体力/攻击力）。
- AI 自动执行回合制战斗，输出格式化的 `[战报]`。
- 命途能力在战斗中自动生效（如繁荣的回血、死亡的反扑）。

---

## 管理员后台

### 权限体系（RBAC）

- 预置权限：`manage_items`, `manage_dungeons`, `manage_users`, `manage_system`, `manage_fate`, `manage_history`, `view_admin`, `admin_access`。
- `root` 拥有全部权限，可分配权限给其他用户。
- 权限管理界面 `/admin/permissions`。

### 主要管理功能

| 页面                 | 功能                                                         |
| -------------------- | ------------------------------------------------------------ |
| `/admin`             | 综合面板，包含物品、分组、副本、用户、系统、命途、历史、权限入口 |
| `/admin/items`       | 物品增删改、拖拽排序、分组管理                               |
| `/admin/dungeons`    | 副本增删改、拖拽排序、自定义奖池（添加/移除物品，调概率/数量） |
| `/admin/users`       | 用户列表、禁言/解禁、设/撤OP、调积分、重置命途、删除用户（级联清理） |
| `/admin/fate`        | 命途树形管理（增删改节点）                                   |
| `/admin/history`     | 副本对话历史（查看、搜索、批量删除）                         |
| `/admin/logs`        | 管理员操作日志（筛选、查看）                                 |
| `/admin/permissions` | 权限分配（仅 root）                                          |

欢迎来到 **无限世界** 的官方维基。

这是一个基于 **Flask + SQLAlchemy + PostgreSQL** 开发的**多人角色扮演互动游戏平台**。玩家可以进入由 AI（DeepSeek）驱动的副本，通过文字冒险推进剧情，获取积分、道具，并沿着独特的**树形命途系统**成长。

---

## 📖 目录

1. [项目简介](#项目简介)
2. [核心功能](#核心功能)
3. [技术架构](#技术架构)
4. [数据库设计](#数据库设计)
5. [命途系统（树形）](#命途系统树形)
6. [副本与AI交互](#副本与ai交互)
7. [管理员后台](#管理员后台)
8. [部署与运维](#部署与运维)
9. [开发与扩展](#开发与扩展)
10. [许可证](#许可证)

---

## 项目简介

**无限世界** 是一个让玩家通过文字与 AI 互动，在随机生成的副本中冒险的 Web 应用。它融合了传统的 MUD 风格、AI 叙事和现代 Web 技术。

- **🎯 核心玩法**：玩家进入副本，与 AI 对话，做出选择，使用道具，战斗，最终通关获得积分和命途经验。
- **🌀 命途系统**：每个玩家通过 25 道测试题绑定一个“命途”（如繁荣、死亡、欺诈），命途以**树形结构**展开，玩家可在达到条件时选择进化分支。
- **👥 多人协作**：支持团队副本，玩家可匹配组队，共享剧情，协同作战。
- **💰 经济系统**：积分、商店、交易市场、背包，构成完整的玩家经济循环。
- **📊 管理后台**：完善的权限体系（RBAC），支持管理物品、副本、用户、命途、历史记录等。

---

## 核心功能

| 功能模块       | 说明                                                    |
| -------------- | ------------------------------------------------------- |
| **用户系统**   | 注册/登录、个人主页（Markdown简介）、头像上传、密码修改 |
| **命途系统**   | 25题测试绑定命途，树形分支进化，能力解锁，红点提醒      |
| **副本系统**   | 单人/团队副本，AI 驱动叙事，道具使用，战斗，通关结算    |
| **商店系统**   | 购买道具，道具分组展示，AI 自动生成描述                 |
| **交易市场**   | 挂单出售，购买，5% 手续费                               |
| **聊天系统**   | 世界频道、私信、团队频道                                |
| **排行榜**     | 积分榜、命途经验榜，支持头像展示                        |
| **管理员后台** | 物品/分组/副本/用户/命途/历史/权限/日志管理             |

---

## 技术架构

### 后端

- **Web 框架**: Flask 3.x
- **ORM**: SQLAlchemy 2.x
- **数据库**: PostgreSQL 14+（生产环境），开发可用 SQLite
- **AI 引擎**: DeepSeek API（v4-flash）
- **任务调度**: 内置定时清理（每天一次）
- **部署**: Gunicorn + PM2（可选）

### 前端

- **语言**: 原生 JavaScript（ES6+）
- **样式**: 纯 CSS（双主题：亮/暗）
- **渲染**: 服务端 Jinja2 模板
- **交互**: Fetch API + Server-Sent Events（流式 AI 回复）

### 部署环境

- **OS**: Ubuntu 22.04 LTS
- **Web Server**: Gunicorn（反向代理可选 Nginx）
- **进程管理**: PM2 或 systemd
- **存储**: 文件系统（头像、静态资源）

---

## 数据库设计

### 核心表（共 31 张）

| 表名                   | 说明                     | 关键字段                                                     |
| ---------------------- | ------------------------ | ------------------------------------------------------------ |
| `user`                 | 用户账号、积分、命途绑定 | `id`, `username`, `coins`, `fate_id`, `current_fate_node_id` |
| `fate`                 | 命途定义（根）           | `id`, `name`, `motto`, `color`, `icon`                       |
| `fate_node`            | 命途树形节点             | `id`, `fate_id`, `parent_id`, `name`, `required_level`, `cost_coins` |
| `user_fate_node`       | 玩家解锁的节点           | `user_id`, `node_id`                                         |
| `fate_question`        | 命途测试题（25题）       | `id`, `question_text`, `option_*`, `weight_*`                |
| `user_fate_answer`     | 玩家答题进度             | `user_id`, `answers`(JSON), `current_question`, `submitted`  |
| `dungeon`              | 副本定义                 | `id`, `name`, `rules`, `is_hidden`, `is_team`, `max_team_size` |
| `dungeon_session`      | 单人副本会话             | `user_id`, `dungeon_id`, `history`(JSON)                     |
| `team`                 | 团队                     | `id`, `dungeon_id`, `member_ids`(JSON)                       |
| `team_session`         | 团队副本会话             | `team_id`, `player_id`, `history`(JSON)                      |
| `team_action`          | 团队公开行动记录         | `team_id`, `player_id`, `description`, `is_private`          |
| `team_chat`            | 团队聊天消息             | `team_id`, `sender_id`, `content`                            |
| `item`                 | 道具定义                 | `id`, `name`, `price`, `effect`, `description`, `group_id`   |
| `item_group`           | 道具分组                 | `id`, `name`                                                 |
| `user_item`            | 玩家背包                 | `user_id`, `item_id`, `quantity`                             |
| `order`                | 交易市场订单             | `id`, `seller_id`, `item_id`, `quantity`, `price_per_unit`, `status` |
| `chat_message`         | 世界聊天                 | `id`, `user_id`, `content`, `timestamp`                      |
| `private_message`      | 私信                     | `sender_id`, `receiver_id`, `content`, `timestamp`           |
| `matching_queue`       | 团队匹配队列             | `user_id`, `dungeon_id`, `status`                            |
| `dungeon_clear_record` | 通关记录（首次奖励）     | `user_id`, `dungeon_id`                                      |
| `dungeon_pool_item`    | 副本自定义奖池           | `dungeon_id`, `item_id`, `quantity_min`, `quantity_max`, `drop_rate` |
| `session_history_log`  | 副本历史存档             | `session_type`, `session_id`, `player_id`, `history`(JSON), `cleared` |
| `admin_log`            | 管理员操作日志           | `admin_id`, `action_type`, `target_type`, `target_id`, `details` |
| `permission`           | 权限定义                 | `id`, `key`, `name`                                          |
| `user_permission`      | 用户权限分配             | `user_id`, `permission_id`                                   |
| ……                     | 其他辅助表               | 详见代码                                                     |

---

## 命途系统（树形）

### 设计理念

每个命途是一个树形结构，根节点为初始能力，后续节点为分支进化。玩家从根节点开始，达到条件后选择解锁子节点，形成**个性化路径**。

### 数据结构

- `FateNode`：节点（id, fate_id, parent_id, name, required_level, cost_coins, …）
- `UserFateNode`：玩家解锁记录
- `User.current_fate_node_id`：玩家当前所在节点

### 玩家流程

1. **觉醒**：完成25题 → 绑定命途 → 自动解锁根节点。
2. **查看路径**：在 `/fate/detail` 查看已走过的线性路径（从根到当前节点）。
3. **红点提醒**：当存在可解锁子节点且玩家等级/积分达标时，显示红点。
4. **进化选择**：点击“进化”进入分支选择界面，展示所有当前节点的子节点，达标节点高亮可点击。
5. **解锁**：消耗积分，解锁新节点，更新当前指针，红点消失（若还有下一层则继续出现）。

### 管理员操作

- 在 `/admin/fate` 管理任意命途的树形结构。
- 支持**添加根节点/子节点**（最多5层）、**编辑**、**删除**（自动提升子节点，重置受影响的玩家）。

---

## 副本与AI交互

### 单人副本

- 玩家进入副本 → 创建/恢复 `DungeonSession`。
- 每次发送消息，后端构建 `system_prompt`（含副本规则、命途能力、奖池列表），调用 DeepSeek API。
- AI 回复以 **流式（SSE）** 返回，前端实时显示。
- **道具使用**：前端点击“使用” → 后端扣除数量 → 消息中标注 `[道具使用]` → AI 描述效果，并输出 `consume: 道具名, 数量` 指令（后端解析扣减）。
- **通关判定**：AI 回复包含 `[通关]`、`[积分:数字]`、可选 `[道具:名称]` → 后端解析，发放奖励（积分、道具、命途经验）。
- **首次通关**额外获得 30 命途经验，重复通关仅积分。

### 团队副本

- 匹配：玩家点击匹配 → 进入队列 → 满员后自动组队。
- 会话：每个队员独立 `TeamSession`，但共享 `TeamAction`（公开行动）和 `TeamChat`。
- AI 提示词包含队友列表和近期公开行动，从**当前玩家视角**叙事。
- 通关奖励：**每个队员独立获得积分**（各自命途加成），各自获得首次通关经验（40点）。
- 道具奖励：只发放给当前玩家。

### 战斗系统（内置在 AI 提示词中）

- 玩家属性由命途等级动态计算（生命/体力/攻击力）。
- AI 自动执行回合制战斗，输出格式化的 `[战报]`。
- 命途能力在战斗中自动生效（如繁荣的回血、死亡的反扑）。

---

## 管理员后台

### 权限体系（RBAC）

- 预置权限：`manage_items`, `manage_dungeons`, `manage_users`, `manage_system`, `manage_fate`, `manage_history`, `view_admin`, `admin_access`。
- `root` 拥有全部权限，可分配权限给其他用户。
- 权限管理界面 `/admin/permissions`。

### 主要管理功能

| 页面                 | 功能                                                         |
| -------------------- | ------------------------------------------------------------ |
| `/admin`             | 综合面板，包含物品、分组、副本、用户、系统、命途、历史、权限入口 |
| `/admin/items`       | 物品增删改、拖拽排序、分组管理                               |
| `/admin/dungeons`    | 副本增删改、拖拽排序、自定义奖池（添加/移除物品，调概率/数量） |
| `/admin/users`       | 用户列表、禁言/解禁、设/撤OP、调积分、重置命途、删除用户（级联清理） |
| `/admin/fate`        | 命途树形管理（增删改节点）                                   |
| `/admin/history`     | 副本对话历史（查看、搜索、批量删除）                         |
| `/admin/logs`        | 管理员操作日志（筛选、查看）                                 |
| `/admin/permissions` | 权限分配（仅 root）                                          |
