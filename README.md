# Emby Account Manager - OpenClaw Skill

An OpenClaw skill for managing Emby media server accounts through Feishu (Lark) group chat.

## Features

- **Account Management**: Register, delete, disable, enable accounts
- **Renewal**: Extend account validity period
- **Password Reset**: Change account passwords
- **Query**: Look up account details
- **Auto Logging**: All operations automatically recorded in Feishu Bitable (multi-dimensional spreadsheet)
- **Permission Control**: Operation logs only visible to group admins

## Supported Commands

| Command | Format | Example |
|---------|--------|---------|
| Register | `注册 <account> <days>` | `注册 18812341234 365` |
| Renew | `续费 <account> <days>` | `续费 18812341234 30` |
| Delete | `删除 <account>` | `删除 13800001111` |
| Password | `改密码 <account> <new_password>` | `改密码 13800001111 mypass` |
| Disable | `禁用 <account>` | `禁用 13800001111` |
| Enable | `启用 <account>` | `启用 13800001111` |
| Query | `查询 <account>` | `查询 13800001111` |
| Config | `配置 Emby API` | Admin only |

### Batch Registration/Renewal

Register or renew multiple accounts at once (one per line):

```
注册
18812341234 365
18898765432 30
13900001111 90
```

## Installation

1. Copy the `emby-account` folder to your OpenClaw workspace skills directory:
   ```bash
   cp -r emby-account ~/.openclaw/workspace/skills/
   ```

2. Create the API config file:
   ```bash
   cat > ~/.openclaw/workspace/emby-config.json << 'EOF'
   {
     "api_base_url": "https://your-emby-server.com/api",
     "api_key": "your-api-key-here"
   }
   EOF
   ```

3. Restart OpenClaw gateway

## Requirements

- OpenClaw v2026.2.26+
- Feishu/Lark plugin (`openclaw-lark`) installed and configured
- Emby account management API server

## Bitable Structure

The skill automatically creates a Feishu Bitable with two tables:

### Renewal Records Table
| Field | Type | Description |
|-------|------|-------------|
| Initiator | Text | Feishu username |
| Time | DateTime | Operation time |
| Account | Text | Emby account |
| Days | Number | Renewal days |
| New Expiry | DateTime | New expiration date |
| Result | Text | Success/failure details |

### Operation Records Table
| Field | Type | Description |
|-------|------|-------------|
| Initiator | Text | Feishu username |
| Time | DateTime | Operation time |
| Type | Select | Register/Delete/Password/Disable/Enable/Query |
| Account | Text | Emby account |
| Valid Days | Number | Only for registration |
| Expiry Date | DateTime | Account expiration |
| Details | Text | API response message |
| Result | Select | Success/Failure |

## Troubleshooting

### `The model did not produce a response before the LLM idle timeout`

Account management tasks chain multiple tool calls (Emby API → Feishu auth → Bitable write), which can exceed OpenClaw's default LLM idle timeout — especially for batch operations or first-time Bitable creation.

Increase `agents.defaults.llm.idleTimeoutSeconds` in your OpenClaw config (typically `~/.openclaw/config.yaml`), or set to `0` to disable the timeout:

```yaml
agents:
  defaults:
    llm:
      idleTimeoutSeconds: 300  # or 0 to disable
```

Restart the OpenClaw gateway for the change to take effect.

### Operation succeeded but Bitable row is missing

This skill is designed so that Bitable write failures **do not block** the main Emby operation. When the skill detects a write failure, it still replies with the Emby result and appends `⚠️ 多维表格写入失败：<reason>`.

If you see this warning, possible causes:
- `app_token` / `table_id` in `~/.openclaw/workspace/emby-bitable.json` is stale (table recreated, token rotated)
- Feishu app lacks write permission on the Bitable
- Field names in the Bitable were renamed manually and no longer match the skill's schema

Fix the underlying issue, then re-log the operation manually if needed.

### Renewal of a non-existent account

When `续费 <account> <days>` targets an unregistered account, the skill auto-registers the account with `<days>` as its initial validity, and writes **two** rows: one to `operation_table` (type: Register) and one to `renewal_table` (with a note indicating auto-registration). This preserves the original "renewal" intent and initiator timestamp for statistics.

## License

MIT
