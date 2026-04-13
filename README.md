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
| Register | `注册 <account>` or `注册 <account> <days>天` | `注册 13800001111 90天` |
| Delete | `删除 <account>` | `删除 13800001111` |
| Renew | `续费 <account> <days>天` | `续费 13800001111 30天` |
| Password | `改密码 <account> <new_password>` | `改密码 13800001111 mypass` |
| Disable | `禁用 <account>` | `禁用 13800001111` |
| Enable | `启用 <account>` | `启用 13800001111` |
| Query | `查询 <account>` | `查询 13800001111` |
| Config | `配置 Emby API` | Admin only |

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

## License

MIT
