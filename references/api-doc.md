# Emby 账号管理 API 参考

## 基本信息

- **Base URL:** 从 skill 配置中读取（由群管理员设置）
- **认证方式:** 请求头 `X-API-Key`
- **数据格式:** JSON

## 统一响应格式

```json
{
  "success": true,
  "message": "操作描述",
  "data": { ... }
}
```

失败时 `success` 为 `false`，HTTP 状态码：`400` 参数错误 / `401` 认证失败 / `404` 不存在 / `409` 已存在

---

## 接口列表

### 1. 注册账号

```
POST /api/accounts/
```

**请求参数:**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| username | string | 是 | 账号名 |
| password | string | 否 | 密码，不传则取账号后6位 |
| valid_days | int | 否 | 有效天数，默认30 |
| expire_date | string | 否 | 到期日期(YYYY-MM-DD)，优先于valid_days |
| remark | string | 否 | 备注 |

**响应示例:**

```json
{
  "success": true,
  "message": "账号 13800001111 创建成功",
  "data": {
    "username": "13800001111",
    "password": "001111",
    "expire_date": "2026-05-13T23:59:59+00:00",
    "task_id": 123
  }
}
```

---

### 2. 查询账号列表

```
GET /api/accounts/
```

**查询参数:**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| status | string | 否 | 筛选状态: active / disabled / expired |
| keyword | string | 否 | 搜索账号名 |
| page | int | 否 | 页码，默认1 |
| page_size | int | 否 | 每页条数，默认20，最大100 |

---

### 3. 查询单个账号

```
GET /api/accounts/{username}/
```

**响应包含:** id, username, status, expire_date, valid_days, disabled_at, remark, created_at, emby_servers

---

### 4. 延期账号（续费）

```
POST /api/accounts/{username}/extend/
```

**请求参数（二选一）:**

| 字段 | 类型 | 说明 |
|------|------|------|
| days | int | 在当前到期时间上增加天数（已过期则从今天算起） |
| expire_date | string | 指定新的到期日期(YYYY-MM-DD) |

如果账号已禁用且延期后未过期，会自动解冻。此接口为同步操作，立即生效。

---

### 5. 修改密码

```
POST /api/accounts/{username}/password/
```

**请求参数:**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| new_password | string | 是 | 新密码 |

---

### 6. 禁用账号

```
POST /api/accounts/{username}/disable/
```

---

### 7. 启用账号

```
POST /api/accounts/{username}/enable/
```

---

### 8. 删除账号

```
DELETE /api/accounts/{username}/
```

删除操作会先删除所有 Emby 服务器上的用户，全部成功后再删除本地记录。

---

## 说明

- 注册、删除、禁用、启用、改密码均为异步操作，返回 `task_id`
- 延期为同步操作，立即生效
- 注册时账号会自动分发到所有启用的 Emby 服务器
- 密码不传时默认取账号名后6位
