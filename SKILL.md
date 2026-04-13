---
name: emby-account
description: |
  Emby 账号管理 skill。通过飞书群聊指令管理 Emby 媒体服务器账号。
  支持注册、删除、续费、改密码、禁用、启用、查询账号。
  所有操作自动记录到飞书多维表格（Bitable）。
metadata:
  openclaw:
    emoji: "🎬"
---

# Emby 账号管理

## 概述

本 skill 用于在飞书群内管理 Emby 媒体服务器账号。用户在群内发送指令，系统自动调用 Emby API 执行操作，并将操作记录写入飞书多维表格。

## API 配置

管理员需要提供以下配置信息，skill 将存储在 workspace 的配置文件中：

- **API Base URL**: Emby 账号管理平台的 API 地址
- **API Key**: 用于 `X-API-Key` 请求头的认证密钥

配置文件路径: `~/.openclaw/workspace/emby-config.json`

```json
{
  "api_base_url": "https://your-server.com/api",
  "api_key": "your-api-key-here"
}
```

如果配置文件不存在，提示管理员先配置。管理员可以说"配置 Emby API"来设置。

## 指令格式

用户在飞书群内发送以下格式的消息触发操作：

| 指令 | 格式 | 示例 | 说明 |
|------|------|------|------|
| 注册 | `注册 <账号>` 或 `注册 <账号> <天数>天` | `注册 13800001111` / `注册 13800001111 90天` | 创建账号，默认30天 |
| 删除 | `删除 <账号>` | `删除 13800001111` | 删除账号（不可恢复） |
| 续费 | `续费 <账号> <天数>天` | `续费 13800001111 30天` | 在当前到期时间上增加天数 |
| 改密码 | `改密码 <账号> <新密码>` | `改密码 13800001111 mypass123` | 修改账号密码 |
| 禁用 | `禁用 <账号>` | `禁用 13800001111` | 禁用账号 |
| 启用 | `启用 <账号>` | `启用 13800001111` | 启用账号 |
| 查询 | `查询 <账号>` | `查询 13800001111` | 查询账号信息 |
| 配置 | `配置 Emby API` | `配置 Emby API` | 管理员设置 API 信息 |

## 执行流程

当收到用户消息时，按以下步骤执行：

### 步骤 1：读取配置

使用 shell 工具读取配置文件：

```bash
cat ~/.openclaw/workspace/emby-config.json
```

如果文件不存在，回复用户："请管理员先配置 Emby API 信息，发送「配置 Emby API」开始配置。"

### 步骤 2：解析指令

从用户消息中识别操作类型和参数：
- 提取操作关键词（注册/删除/续费/改密码/禁用/启用/查询/配置）
- 提取账号名（通常是手机号或用户名）
- 提取附加参数（天数、密码等）

### 步骤 3：调用 Emby API

根据操作类型，使用 `fetch` 工具调用对应的 API：

#### 注册账号
```
fetch("POST", "{api_base_url}/accounts/", {
  headers: { "X-API-Key": "{api_key}", "Content-Type": "application/json" },
  body: { "username": "<账号>", "valid_days": <天数> }
})
```

#### 查询账号
```
fetch("GET", "{api_base_url}/accounts/<账号>/", {
  headers: { "X-API-Key": "{api_key}" }
})
```

#### 续费账号
```
fetch("POST", "{api_base_url}/accounts/<账号>/extend/", {
  headers: { "X-API-Key": "{api_key}", "Content-Type": "application/json" },
  body: { "days": <天数> }
})
```

#### 修改密码
```
fetch("POST", "{api_base_url}/accounts/<账号>/password/", {
  headers: { "X-API-Key": "{api_key}", "Content-Type": "application/json" },
  body: { "new_password": "<新密码>" }
})
```

#### 禁用账号
```
fetch("POST", "{api_base_url}/accounts/<账号>/disable/", {
  headers: { "X-API-Key": "{api_key}" }
})
```

#### 启用账号
```
fetch("POST", "{api_base_url}/accounts/<账号>/enable/", {
  headers: { "X-API-Key": "{api_key}" }
})
```

#### 删除账号
```
fetch("DELETE", "{api_base_url}/accounts/<账号>/", {
  headers: { "X-API-Key": "{api_key}" }
})
```

### 步骤 4：记录到多维表格

每次操作完成后，将记录写入飞书多维表格。

#### 首次使用：创建多维表格

如果 `~/.openclaw/workspace/emby-bitable.json` 不存在，需要初始化：

1. **创建多维表格**：使用 `feishu_bitable_app` 工具，action=`create`
   ```json
   { "action": "create", "name": "Emby账号管理记录" }
   ```
   保存返回的 `app_token`

2. **创建续费记录表**：使用 `feishu_bitable_app_table` 工具，action=`create`
   ```json
   {
     "action": "create",
     "app_token": "<app_token>",
     "table": { "name": "续费记录" }
   }
   ```

3. **为续费记录表创建字段**：使用 `feishu_bitable_app_table_field` 工具，action=`create`，依次创建：
   - `发起人`（类型 1，文本）
   - `发起时间`（类型 5，日期时间）
   - `账户名`（类型 1，文本）
   - `续费天数`（类型 2，数字）
   - `新到期日期`（类型 5，日期时间）
   - `操作结果`（类型 1，文本）

4. **创建操作记录表**：同上方式创建表，名称为"操作记录"

5. **为操作记录表创建字段**：
   - `发起人`（类型 1，文本）
   - `发起时间`（类型 5，日期时间）
   - `操作类型`（类型 3，单选，选项：注册/删除/改密码/禁用/启用/查询）
   - `账户名`（类型 1，文本）
   - `有效天数`（类型 2，数字）
   - `到期日期`（类型 5，日期时间）
   - `操作详情`（类型 1，文本）
   - `操作结果`（类型 3，单选，选项：成功/失败）

6. **保存配置**到 `~/.openclaw/workspace/emby-bitable.json`：
   ```json
   {
     "app_token": "<app_token>",
     "renewal_table_id": "<续费表table_id>",
     "operation_table_id": "<操作表table_id>"
   }
   ```

7. **设置权限**：使用 `feishu_drive_file` 工具设置多维表格仅群管理员可查看

#### 写入记录

读取 `~/.openclaw/workspace/emby-bitable.json` 获取 `app_token` 和 `table_id`。

**续费操作** → 写入"续费记录"表：
```json
{
  "action": "create",
  "app_token": "<app_token>",
  "table_id": "<renewal_table_id>",
  "fields": {
    "发起人": "<飞书用户名>",
    "发起时间": <当前时间戳毫秒>,
    "账户名": "<账号>",
    "续费天数": <天数>,
    "新到期日期": <新到期时间戳毫秒>,
    "操作结果": "成功/失败详情"
  }
}
```

**其他操作**（注册/删除/改密码/禁用/启用/查询） → 写入"操作记录"表：
```json
{
  "action": "create",
  "app_token": "<app_token>",
  "table_id": "<operation_table_id>",
  "fields": {
    "发起人": "<飞书用户名>",
    "发起时间": <当前时间戳毫秒>,
    "操作类型": "<操作类型>",
    "账户名": "<账号>",
    "有效天数": <天数或null>,
    "到期日期": <到期时间戳毫秒或null>,
    "操作详情": "<API返回的message>",
    "操作结果": "成功/失败"
  }
}
```

### 步骤 5：回复用户

在飞书群内回复操作结果，格式简洁明了：

**注册成功：**
> ✅ 账号 13800001111 注册成功
> 密码：001111
> 有效期至：2026-05-13

**续费成功：**
> ✅ 账号 13800001111 续费成功
> 续费 30 天，新到期日期：2026-06-12

**查询结果：**
> 📋 账号：13800001111
> 状态：active
> 到期日期：2026-05-13
> 剩余天数：30 天
> 服务器：G（123）✅, N ✅

**操作失败：**
> ❌ 操作失败：<错误信息>

## 管理员配置流程

当管理员发送"配置 Emby API"时：

1. 询问 API 地址（Base URL）
2. 询问 API Key
3. 将配置写入 `~/.openclaw/workspace/emby-config.json`
4. 回复配置成功

## 安全注意事项

- API Key 存储在本地配置文件中，不在群内展示
- 密码信息仅在注册时回复给操作者，不记录在多维表格中
- 多维表格仅群管理员可查看
- 删除操作需要确认（回复"确认删除 <账号>"后执行）

## 错误处理

- API 返回 401：提示 API Key 无效，请管理员重新配置
- API 返回 404：提示账号不存在
- API 返回 409：提示账号已存在（注册时）
- 网络错误：提示网络异常，请稍后重试
- 指令格式错误：提示正确的指令格式
