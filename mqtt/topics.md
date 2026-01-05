# MQTT 主題規範

> **唯一真相來源** - 基於 ESP32 韌體 V3.0 (iotWork_V3.0)
> 
> ⚠️ **所有專案必須嚴格遵守此規範，禁止自創主題或修改格式**

## 主題列表

### 設備數據（ESP32 → Server）

| 主題格式 | 用途 | 訊息格式 | QoS | Retain |
|----------|------|----------|-----|--------|
| `device/{chip_id}/data/update` | 遊戲數據上報 | JSON | 0 | ❌ |
| `device/{chip_id}/status` | 設備狀態 | 純文字 | 0 | ✅ |
| `device/{chip_id}/config/timezone` | 時區更新通知 | JSON | 0 | ❌ |
| `device/{chip_id}/events/daily_reset` | 每日歸零通知 | JSON | 0 | ❌ |

### 認證（雙向）

| 主題格式 | 方向 | 用途 | QoS |
|----------|------|------|-----|
| `device/{chip_id}/auth/request` | ESP32 → Server | 認證請求 | 0 |
| `device/{chip_id}/auth/response` | Server → ESP32 | 認證回應 | 0 |

### 指令（Server → ESP32）

| 主題格式 | 用途 | QoS |
|----------|------|-----|
| `device/{chip_id}/command` | 指令下發（開分、洗分等） | 0 |

### LWT (Last Will and Testament)

設備連線時會設定遺囑訊息：
- **主題**: `device/{chip_id}/status`
- **訊息**: `offline`
- **Retain**: `true`

當設備異常斷線時，Broker 會自動發布此訊息。

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

**欄位說明：**
| 欄位 | 類型 | 說明 |
|------|------|------|
| `credit_in` | uint32 | 入金計數 |
| `coin_out` | uint32 | 出金計數 |
| `ball_in` | uint32 | 入球計數 |
| `ball_out` | uint32 | 出球計數 |
| `assign_credit` | uint32 | 開分計數 |
| `settled_credit` | uint32 | 洗分計數 |
| `bill_denomination` | uint32 | 紙鈔面額 |
| `last_activity` | ulong | 最後活動時間戳 |

### 設備狀態 (`device/{chip_id}/status`)

純文字：`online` 或 `offline`

- **上線時**: 設備主動發布 `online`（retain=true）
- **離線時**: Broker 透過 LWT 自動發布 `offline`（retain=true）

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
# infra/listener.py 必須訂閱以下主題
MQTT_TOPICS = [
    ("device/+/data/update", 0),      # 遊戲數據
    ("device/+/status", 0),           # 設備狀態
    ("device/+/auth/request", 0),     # 認證請求
    ("device/+/config/timezone", 0),  # 時區變更
    ("device/+/events/daily_reset", 0), # 每日歸零
]
```

## 欄位對照表（韌體 → 後端）

> ⚠️ **重要**: 韌體發送的欄位名稱與後端資料庫欄位不同，需要轉換

| 韌體欄位 (MQTT) | 後端欄位 (DB) | 說明 |
|-----------------|---------------|------|
| `credit_in` | `coin_in_count` | 入金計數 |
| `coin_out` | `payout_count` | 出金計數 |
| `ball_in` | `ball_in_count` | 入球計數 |
| `ball_out` | `ball_out_count` | 出球計數 |
| `assign_credit` | `assign_count` | 開分計數 |
| `settled_credit` | `settled_count` | 洗分計數 |

**轉換責任**: `infra/listener.py` 負責將韌體欄位轉換為後端欄位

## 各專案實作要求

### ESP32 韌體 (iotWork_V3.0)
- ✅ 發布數據使用上述 JSON 格式
- ✅ 連線時設定 LWT
- ✅ 上線時發布 `online` 狀態

### infra (listener.py)
- ⚠️ 訂閱所有 `device/+/*` 主題
- ⚠️ 欄位名稱轉換（韌體 → 後端）
- ⚠️ 處理認證請求並回應

### iot.tg25 (Laravel)
- ⚠️ 接收 listener 轉發的數據
- ⚠️ 使用後端欄位名稱存入資料庫

---
*來源：iotWork_V3.0/src/modules/MQTTModule.cpp*
*最後更新：2026-01-05*
