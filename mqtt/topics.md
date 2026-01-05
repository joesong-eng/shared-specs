# MQTT 主題規範

> **唯一真相來源** - ESP32 韌體與後端必須使用相同主題

## 主題列表

### v6 格式（ESP32 實際使用）

| 用途 | 主題格式 | 發布者 | 訂閱者 |
|------|----------|--------|--------|
| 設備數據上報 | `device/{chip_id}/data/update` | ESP32 | infra (listener.py) |
| 設備狀態 | `device/{chip_id}/status` | ESP32 | infra (listener.py) |

### v7 格式（模擬器使用）

| 用途 | 主題格式 | 發布者 | 訂閱者 |
|------|----------|--------|--------|
| 設備數據上報 | `iot/data/{device_id}` | 模擬器 | infra (listener.py) |

> ⚠️ **注意**：v6 和 v7 格式並存，listener.py 同時支援兩種格式

## 訊息格式

### v6 格式 - 設備數據 (`device/{chip_id}/data/update`)

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

### v6 格式 - 設備狀態 (`device/{chip_id}/status`)

純文字：`online` 或 `offline`

### v7 格式 - 設備數據 (`iot/data/{device_id}`)

```json
{
  "coin_in_count": 100,
  "payout_count": 50,
  "ball_in_count": 0,
  "ball_out_count": 0,
  "assign_count": 0,
  "settled_count": 0,
  "timestamp": "2026-01-05T10:30:00Z",
  "data_type": "periodic"
}
```

## 欄位對照表

| v6 欄位 | v7 欄位 | 說明 |
|---------|---------|------|
| credit_in | coin_in_count | 入金計數 |
| coin_out | payout_count | 出金計數 |
| ball_in | ball_in_count | 入球計數 |
| ball_out | ball_out_count | 出球計數 |
| assign_credit | assign_count | 開分計數 |
| settled_credit | settled_count | 洗分計數 |

## 修改流程

1. 修改此文件
2. 同步更新 listener.py
3. 如需修改 ESP32 韌體，需重新燒錄

---
*最後更新：2026-01-05*
