# API 契約規範

> **唯一真相來源** - 跨專案 API 呼叫必須依據此規範

## 設備認證流程

### 流程圖

```
ESP32                          listener.py                    Database
  │                                │                              │
  │─── MQTT connect ──────────────>│                              │
  │                                │                              │
  │─── auth/request ──────────────>│                              │
  │    {chip_id, auth_key,         │                              │
  │     firmware_version}          │                              │
  │                                │─── 查詢 auth_key ───────────>│
  │                                │<── 驗證結果 ─────────────────│
  │                                │                              │
  │<── auth/response ─────────────│                              │
  │    {status: success/failed}    │                              │
  │                                │                              │
  │ [success] → STATE_OPERATIONAL  │                              │
  │ [failed]  → STATE_ERROR        │                              │
```

### 認證請求 (ESP32 → Server)

| 項目 | 值 |
|------|-----|
| 主題 | `device/{chip_id}/auth/request` |
| 發布者 | ESP32 韌體 |
| 時機 | MQTT 連線成功後立即發送 |

**Request:**
```json
{
  "chip_id": "iot_001",
  "auth_key": "xxx",
  "firmware_version": "v3.0.0"
}
```

### 認證回應 (Server → ESP32)

| 項目 | 值 |
|------|-----|
| 主題 | `device/{chip_id}/auth/response` |
| 發布者 | infra (listener.py) |

**⚠️ 重要：listener.py 必須驗證 auth_key**

驗證邏輯：
1. 收到 `auth/request` 後，查詢資料庫
2. 檢查 `chip_id` 是否存在
3. 檢查 `auth_key` 是否匹配
4. 回應驗證結果

**成功回應：**
```json
{
  "status": "success"
}
```

**失敗回應：**
```json
{
  "status": "failed",
  "message": "Invalid auth_key"
}
```

### ESP32 狀態轉換

| 回應 | ESP32 狀態 | 說明 |
|------|------------|------|
| `status: "success"` | `STATE_OPERATIONAL` | 正常運作，可收發數據 |
| `status: "failed"` 或其他 | `STATE_ERROR` | 錯誤狀態，無法正常運作 |

### listener.py 實作範例

```python
async def handle_auth_request(chip_id: str, payload: dict):
    """處理設備認證請求"""
    auth_key = payload.get("auth_key")
    firmware_version = payload.get("firmware_version")
    
    # 查詢資料庫驗證
    device = await db.query(
        "SELECT * FROM devices WHERE chip_id = %s AND auth_key = %s",
        (chip_id, auth_key)
    )
    
    response_topic = f"device/{chip_id}/auth/response"
    
    if device:
        # 更新韌體版本和上線狀態
        await db.execute(
            "UPDATE devices SET firmware_version = %s, is_online = TRUE WHERE chip_id = %s",
            (firmware_version, chip_id)
        )
        await mqtt.publish(response_topic, {"status": "success"})
    else:
        await mqtt.publish(response_topic, {
            "status": "failed",
            "message": "Invalid chip_id or auth_key"
        })
```

---

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

---
*最後更新：2026-01-06*
