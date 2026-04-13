---
name: emby-account
description: |
  Emby账号管理。飞书群内指令：注册/续费/删除/改密码/禁用/启用/查询/统计。
metadata:
  openclaw:
    emoji: "🎬"
---

# Emby 账号管理

飞书群内发送指令管理Emby账号，操作记录写入飞书多维表格。

## 配置

读取 `~/.openclaw/workspace/emby-config.json`：
```json
{"api_base_url": "https://xxx/api", "api_key": "xxx"}
```
不存在则提示管理员发送"配置 Emby API"设置。

多维表格配置 `~/.openclaw/workspace/emby-bitable.json`：
```json
{"app_token": "xxx", "renewal_table_id": "xxx", "operation_table_id": "xxx"}
```

## 指令

| 指令 | 格式 | 说明 |
|------|------|------|
| 注册 | `注册 <账号> [天数]` | 默认30天 |
| 续费 | `续费 <账号> <天数>` | 必须指定天数 |
| 删除 | `删除 <账号>` | 不可恢复 |
| 改密码 | `改密码 <账号> <新密码>` | |
| 禁用/启用 | `禁用/启用 <账号>` | |
| 查询 | `查询 <账号>` | |
| 统计 | `统计 本周/本月/上个月/N月` | 管理员用 |

天数可带"天"字（`365天`→自动去掉→`365`）。

批量注册/续费：第一行只写指令，后续每行`<账号> <天数>`：
```
注册
18812341234 365
18898765432 30
```

## API调用

Headers: `X-API-Key: {api_key}`, `Content-Type: application/json`

- 注册: `POST {base}/accounts/` body `{"username":"x","valid_days":N}`
- 续费: `POST {base}/accounts/{user}/extend/` body `{"days":N}`
- 改密码: `POST {base}/accounts/{user}/password/` body `{"new_password":"x"}`
- 禁用: `POST {base}/accounts/{user}/disable/`
- 启用: `POST {base}/accounts/{user}/enable/`
- 删除: `DELETE {base}/accounts/{user}/`
- 查询: `GET {base}/accounts/{user}/`

## 写多维表格

以发送指令的用户飞书名称为发起人写入。

续费→续费记录表（发起人/发起时间/账户名/续费天数/新到期日期/操作结果）
其他→操作记录表（发起人/发起时间/操作类型/账户名/有效天数/到期日期/操作详情/操作结果）

首次使用如`emby-bitable.json`不存在，用`feishu_bitable_app`创建多维表格"Emby账号管理记录"，建两个表和字段，保存到`emby-bitable.json`，权限设为所有人可编辑。

## 回复格式

注册: `✅ 账号xxx注册成功 密码：xxx 有效期：N天 到期：YYYY-MM-DD`
续费: `✅ 账号xxx续费成功 续费N天 新到期：YYYY-MM-DD`
批量: 每个账号一行 ✅/❌
查询: 账号/状态/到期/剩余天数/服务器

## 统计格式（严格遵守）

```
📊 Emby统计（YYYY年M月）
【续费】
高 → 续费365天 → 5个
高 → 续费30天 → 12个
【注册】
高 → 注册 → 10个
【其他】
改密码：3次 | 禁用：2次 | 启用：1次 | 删除：0次
```
按发起人+天数分组。不列明细、不列账号、不列总用户数。

## 错误处理

401→API Key无效 | 404→账号不存在 | 409→账号已存在
