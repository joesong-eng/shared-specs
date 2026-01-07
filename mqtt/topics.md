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
| `device/{chip_id}/events/alarm` | 擺錘警報通知 | JSON | 0 | ❌ |

### 認證（雙向）

| 主題格式 | 方向 | 用途 | QoS |
|----------|------|------|-----|
| `device/{chip_id}/auth/request` | ESP32 → Server | 認證請求 | 0 |
| `device/{chip_id}/auth/response` | Server → ESP32 | 認證回應 | 0 |

### 指令（Server → ESP32）

| 主題格式 | 用途 | QoS |
|----------|------|-----|
| `device/{chip_id}/command` | 指令下發（開分、洗分、歸零） | 0 |
| `device/{chip_id}/command/response` | 指令執行回應 | 0 |

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
  "credit_out": 50,
  "assign_credit": 0,
  "settled_credit": 0,
  "last_activity": 0
}
```

**欄位說明：**
| 欄位 | 類型 | 說明 |
|------|------|------|
| `credit_in` | uint32 | 入金計數（投幣/開分時計數器跳動次數） |
| `credit_out` | uint32 | 出金計數（洗分時計數器跳動次數） |
| `assign_credit` | uint32 | 開分計數（遠端開分觸發次數） |
| `settled_credit` | uint32 | 洗分計數（遠端洗分觸發次數） |
| `last_activity` | ulong | 最後活動時間戳 |

### 設備狀態 (`device/{chip_id}/status`)

純文字：`online` 或 `offline`

- **上線時**: 設備主動發布 `online`（retain=true）
- **離線時**: Broker 透過 LWT 自動發布 `offline`（retain=true）

### 認證請求 (`device/{chip_id}/auth/request`)

```json
{
  "chip_id": "iot_001",
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

### 擺錘警報 (`device/{chip_id}/events/alarm`)

```json
{
  "event_type": "alarm_triggered",
  "timestamp": 123456789,
  "alarm_type": "tilt",
  "status": "triggered",
  "device_id": "iot_001"
}
```

**欄位說明：**
| 欄位 | 類型 | 說明 |
|------|------|------|
| `event_type` | string | `alarm_triggered` 或 `alarm_cleared` |
| `timestamp` | ulong | 事件發生時間戳 |
| `alarm_type` | string | 警報類型：`tilt`（擺錘傾斜） |
| `status` | string | `triggered`（觸發）或 `cleared`（解除） |
| `device_id` | string | 設備 ID |

## 指令格式 (Server → ESP32)

### 指令請求 (`device/{chip_id}/command`)

所有指令都遵循統一的 JSON 格式：

```json
{
  "command": "assign_credit | settle_credit | reset_counter",
  "transaction_id": "uuid-string",
  "timestamp": 1704499200,
  "params": {
    // 指令專用參數（可選）
  }
}
```

**必要欄位：**
| 欄位 | 類型 | 說明 |
|------|------|------|
| `command` | string | 指令類型：`assign_credit`、`settle_credit`、`reset_counter` |
| `transaction_id` | string | 唯一交易識別符（UUID 格式） |
| `timestamp` | ulong | 指令發送時間戳（Unix timestamp） |

**可選欄位 (`params`)：**
| 欄位 | 類型 | 預設值 | 說明 |
|------|------|--------|------|
| `count` | uint8 | 1 | 開分次數（僅 `assign_credit` 使用） |
| `pulse_duration` | uint16 | 100 | 脈衝時長（毫秒） |
| `pulse_interval` | uint16 | 200 | 脈衝間隔（毫秒，多次開分時使用） |

### 指令回應 (`device/{chip_id}/command/response`)

#### 成功回應

```json
{
  "transaction_id": "uuid-string",
  "command_type": "assign_credit",
  "status": "success",
  "timestamp": 1704499201,
  "data": {
    // 指令專用回應資料
  }
}
```

#### 錯誤回應

```json
{
  "transaction_id": "uuid-string",
  "command_type": "assign_credit",
  "status": "error",
  "timestamp": 1704499201,
  "error": {
    "code": 8001,
    "message": "Invalid JSON format"
  }
}
```

**回應欄位說明：**
| 欄位 | 類型 | 說明 |
|------|------|------|
| `transaction_id` | string | 原始指令的交易識別符 |
| `command_type` | string | 指令類型 |
| `status` | string | `success` 或 `error` |
| `timestamp` | ulong | 回應生成時間戳 |
| `data` | object | 成功時的額外資料（可選） |
| `error` | object | 失敗時的錯誤資訊 |

### 錯誤碼對照表

| 錯誤碼 | 名稱 | 說明 |
|--------|------|------|
| 8001 | `INVALID_JSON` | JSON 解析失敗 |
| 8002 | `MISSING_FIELD` | 缺少必要欄位（command、transaction_id、timestamp） |
| 8003 | `UNKNOWN_COMMAND` | 未知的指令類型 |
| 8004 | `INVALID_PARAMETER` | 參數值無效（如 count 超出範圍） |
| 8005 | `EXECUTION_TIMEOUT` | 指令執行超時 |
| 8006 | `DEVICE_BUSY` | 設備忙碌中，正在執行其他指令 |
| 8007 | `INVALID_STATE` | 設備狀態不允許此操作 |
| 8008 | `RELAY_FAILED` | 繼電器操作失敗 |
| 8009 | `RESET_FAILED` | 計數器歸零失敗 |

## 指令範例

### 開分指令 (`assign_credit`)

**請求：**
```json
{
  "command": "assign_credit",
  "transaction_id": "550e8400-e29b-41d4-a716-446655440001",
  "timestamp": 1704499200,
  "params": {
    "count": 3,
    "pulse_duration": 100,
    "pulse_interval": 200
  }
}
```

**成功回應：**
```json
{
  "transaction_id": "550e8400-e29b-41d4-a716-446655440001",
  "command_type": "assign_credit",
  "status": "success",
  "timestamp": 1704499201,
  "data": {
    "executed_count": 3
  }
}
```

**失敗回應（設備忙碌）：**
```json
{
  "transaction_id": "550e8400-e29b-41d4-a716-446655440001",
  "command_type": "assign_credit",
  "status": "error",
  "timestamp": 1704499201,
  "error": {
    "code": 8006,
    "message": "Device is busy executing previous command"
  }
}
```

### 洗分指令 (`settle_credit`)

**請求：**
```json
{
  "command": "settle_credit",
  "transaction_id": "550e8400-e29b-41d4-a716-446655440002",
  "timestamp": 1704499300
}
```

**成功回應：**
```json
{
  "transaction_id": "550e8400-e29b-41d4-a716-446655440002",
  "command_type": "settle_credit",
  "status": "success",
  "timestamp": 1704499301,
  "data": {}
}
```

### 歸零指令 (`reset_counter`)

**請求：**
```json
{
  "command": "reset_counter",
  "transaction_id": "550e8400-e29b-41d4-a716-446655440003",
  "timestamp": 1704499400
}
```

**成功回應：**
```json
{
  "transaction_id": "550e8400-e29b-41d4-a716-446655440003",
  "command_type": "reset_counter",
  "status": "success",
  "timestamp": 1704499401,
  "data": {
    "pre_reset": {
      "credit_in": 1000,
      "credit_out": 500,
      "assign_credit": 50,
      "settled_credit": 30
    },
    "post_reset": {
      "credit_in": 0,
      "credit_out": 0,
      "assign_credit": 0,
      "settled_credit": 0
    }
  }
}
```

**失敗回應（歸零失敗）：**
```json
{
  "transaction_id": "550e8400-e29b-41d4-a716-446655440003",
  "command_type": "reset_counter",
  "status": "error",
  "timestamp": 1704499401,
  "error": {
    "code": 8009,
    "message": "Failed to persist reset data to NVS"
  }
}
```

### 錯誤範例

**無效 JSON：**
```json
// 請求（格式錯誤）
{ "command": "assign_credit", transaction_id: }

// 回應
{
  "transaction_id": "",
  "command_type": "unknown",
  "status": "error",
  "timestamp": 1704499500,
  "error": {
    "code": 8001,
    "message": "Invalid JSON format"
  }
}
```

**缺少必要欄位：**
```json
// 請求（缺少 transaction_id）
{
  "command": "assign_credit",
  "timestamp": 1704499600
}

// 回應
{
  "transaction_id": "",
  "command_type": "assign_credit",
  "status": "error",
  "timestamp": 1704499601,
  "error": {
    "code": 8002,
    "message": "Missing required field: transaction_id"
  }
}
```

**未知指令：**
```json
// 請求
{
  "command": "unknown_command",
  "transaction_id": "550e8400-e29b-41d4-a716-446655440004",
  "timestamp": 1704499700
}

// 回應
{
  "transaction_id": "550e8400-e29b-41d4-a716-446655440004",
  "command_type": "unknown_command",
  "status": "error",
  "timestamp": 1704499701,
  "error": {
    "code": 8003,
    "message": "Unknown command type: unknown_command"
  }
}
```

## Listener 訂閱設定

```python
# infra/listener.py 必須訂閱以下主題
MQTT_TOPICS = [
    ("device/+/data/update", 0),           # 遊戲數據
    ("device/+/status", 0),                # 設備狀態
    ("device/+/auth/request", 0),          # 認證請求
    ("device/+/config/timezone", 0),       # 時區變更
    ("device/+/events/daily_reset", 0),    # 每日歸零
    ("device/+/events/alarm", 0),          # 擺錘警報
    ("device/+/command/response", 0),      # 指令回應
]
```

## 欄位對照表（韌體 → 後端）

> ⚠️ **重要**: 韌體發送的欄位名稱與後端資料庫欄位不同，需要轉換

| 韌體/MQTT 欄位 | 後端 DB 欄位 | 說明 |
|----------------|--------------|------|
| `credit_in` | `coin_in_count` | 入金計數 |
| `credit_out` | `payout_count` | 出金計數 |
| `assign_credit` | `assign_count` | 開分計數 |
| `settled_credit` | `settled_count` | 洗分計數 |

**轉換責任**: `infra/listener.py` 負責將 MQTT 欄位轉換為 DB 欄位

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
*最後更新：2026-01-06*
