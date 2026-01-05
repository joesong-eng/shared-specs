# 命名慣例

> **唯一真相來源** - 跨專案命名必須一致

## 設備識別碼

| 名稱 | 格式 | 範例 | 說明 |
|------|------|------|------|
| `chip_id` | 自訂字串 | `iot_001` | ESP32 設備 ID，用於 MQTT 主題 |
| `device_uuid` | `{品牌}-{日期}-{序號}` | `WAW-20250130-001` | 後端資料庫唯一識別碼 |

## MQTT 主題命名

| 格式 | 範例 |
|------|------|
| `device/{chip_id}/data/update` | `device/iot_001/data/update` |
| `device/{chip_id}/status` | `device/iot_001/status` |
| `device/{chip_id}/auth/request` | `device/iot_001/auth/request` |
| `device/{chip_id}/auth/response` | `device/iot_001/auth/response` |
| `device/{chip_id}/command` | `device/iot_001/command` |
| `device/{chip_id}/config/timezone` | `device/iot_001/config/timezone` |
| `device/{chip_id}/events/daily_reset` | `device/iot_001/events/daily_reset` |

## 環境變數命名

| 前綴 | 用途 | 範例 |
|------|------|------|
| `MQTT_` | MQTT 相關設定 | `MQTT_HOST`, `MQTT_PORT`, `MQTT_USERNAME` |
| `REDIS_` | Redis 相關設定 | `REDIS_HOST`, `REDIS_PORT` |
| `DB_` | 資料庫相關設定 | `DB_HOST`, `DB_DATABASE` |
| `INTERNAL_` | 內部 API 設定 | `INTERNAL_API_KEY` |

## 韌體版本命名

| 格式 | 範例 | 說明 |
|------|------|------|
| `v{major}.{minor}.{patch}` | `v3.0.0` | 語意化版本 |

## 時區 ID 命名

使用 IANA 時區資料庫格式：

| 時區 ID | 顯示名稱 | UTC 偏移 |
|---------|----------|----------|
| `Asia/Taipei` | 台灣標準時間 | +08:00 |
| `Asia/Jakarta` | 印尼雅加達時間 | +07:00 |
| `Asia/Manila` | 菲律賓標準時間 | +08:00 |

---
*最後更新：2026-01-06*
