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
    "payout_count": 50
  }
}
```

**Headers:**
```
X-Internal-Key: {內部 API 金鑰}
```

---
*最後更新：2026-01-05*
