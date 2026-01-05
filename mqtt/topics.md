# MQTT 主題規範

> **唯一真相來源** - 基於 ESP32 韌體 V3.0 (iotWork_V3.0)

## 主題列表

### 設備數據（ESP32 → Server）

| 主題格式 | 用途 | 訊息格式 |
|----------|------|----------|
| `device/{chip_id}/data/update` | 遊戲數據上報 | JSON |
| `device/{chip_id}/status` | 設備狀態 | 純文字 `online`/`offline` |
| `device/{chip_id}/config/timezone` | 時區更新通知 | JSON |
| `device/{chip_id}/events/daily_reset` | 每日歸零通知 | JSON |

### 認證（雙向）

| 主題格式 | 方向 | 用途 |
|----------|------|------|
| `device/{chip_id}/auth/request` | ESP32 → Server | 認證請求 |
| `device/{chip_id}/auth/response` | Server → ESP32 | 認證回應 |

### 指令（Server → ESP32）

| 主題格式 | 用途 |
|----------|------|
| `device/{chip_id}/command` | 指令下發（開分、洗分等） |

## 訊息格式

### 數據上報 (`device/{chip_id}/data/update`)

```json
{
  "credit_in": 100,
  "coin_out": 50,
  "ball_in": 0,
  "ball_out": 0,
  "assign_credit": 0,
  "settled_credit": 0,
  "bill_denomination": 0,
  "last_activity": 0
}
```

### 設備狀態 (`device/{chip_id}/status`)

純文字：`online` 或 `offline`（使用 retain 標誌）

### 認證請求 (`device/{chip_id}/auth/request`)

```json
{
  "chip_id": "iot_001",
  "auth_key": "xxx",
  "firmware_version": "v3.0.0"
}
```

### 認證回應 (`device/{chip_id}/auth/response`)

```json
{
  "status": "success"
}
```

### 時區更新 (`device/{chip_id}/config/timezone`)

```json
{
  "event_type": "timezone_changed",
  "timestamp": 123456789,
  "timezone": {
    "timezone_id": "Asia/Taipei",
    "display_name": "台灣標準時間",
    "offset_hours": 8,
    "offset_minutes": 0,
    "auto_dst": false,
    "ntp_server": "pool.ntp.org",
    "backup_ntp": "time.windows.com"
  }
}
```

### 每日歸零 (`device/{chip_id}/events/daily_reset`)

```json
{
  "event_type": "daily_reset",
  "timestamp": 123456789,
  "reset_date": "2026-01-05",
  "success": true,
  "message": "Daily reset completed successfully",
  "device_id": "iot_001"
}
```

## Listener 訂閱設定

```python
MQTT_TOPICS = [
    ("device/+/data/update", 0),
    ("device/+/status", 0),
    ("device/+/auth/request", 0),
    ("device/+/config/timezone", 0),
    ("device/+/events/daily_reset", 0),
]
```

## 欄位對照表（韌體 → 後端）

| 韌體欄位 | 後端欄位 | 說明 |
|----------|----------|------|
| credit_in | coin_in_count | 入金計數 |
| coin_out | payout_count | 出金計數 |
| ball_in | ball_in_count | 入球計數 |
| ball_out | ball_out_count | 出球計數 |
| assign_credit | assign_count | 開分計數 |
| settled_credit | settled_count | 洗分計數 |

---
*來源：iotWork_V3.0/src/modules/MQTTModule.cpp*
*最後更新：2026-01-05*
