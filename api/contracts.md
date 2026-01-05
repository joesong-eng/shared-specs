# API 契約規範

> **唯一真相來源** - 跨專案 API 呼叫必須依據此規範

## 內部 API

### 廣播設備更新

| 項目 | 值 |
|------|-----|
| URL | `POST /api/internal/broadcast/device-update` |
| 呼叫者 | infra (listener.py) |
| 接收者 | iot.tg25 (Laravel) |

**Request:**
```json
{
  "device_id": "device_uuid",
  "data": {
    "coin_in_count": 100,
    "payout_count": 50,
    "ball_in_count": 0,
    "ball_out_count": 0,
    "assign_count": 0,
    "settled_count": 0
  }
}
```

**Headers:**
```
X-Internal-Key: {內部 API 金鑰}
```

### 認證回應

| 項目 | 值 |
|------|-----|
| 方式 | MQTT 發布 |
| 主題 | `device/{chip_id}/auth/response` |
| 發布者 | infra (listener.py) |

**Response:**
```json
{
  "status": "success"
}
```

或失敗時：
```json
{
  "status": "failed",
  "message": "Invalid auth_key"
}
```

---
*最後更新：2026-01-06*
