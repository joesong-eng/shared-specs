# 資料庫 Schema 規範

> **唯一真相來源** - 所有專案的資料庫操作必須依據此規範

## 資料庫清單

| 資料庫 | 位置 | 用途 |
|--------|------|------|
| `iot_db` | infra VPS | 設備、場地、交易資料 |

## 核心表格

### devices - 設備表

| 欄位 | 類型 | 說明 |
|------|------|------|
| `id` | BIGINT | 主鍵 |
| `device_uuid` | VARCHAR(50) | 設備唯一識別碼 |
| `chip_id` | VARCHAR(50) | ESP32 晶片 ID |
| `mqtt_topic` | VARCHAR(100) | MQTT 主題前綴 |
| `device_type` | VARCHAR(50) | 設備類型 |
| `firmware_version` | VARCHAR(20) | 韌體版本 |
| `auth_key` | VARCHAR(100) | 認證金鑰（保留欄位，目前未使用） |
| `is_online` | BOOLEAN | 是否在線 |
| `last_heartbeat` | TIMESTAMP | 最後心跳時間 |
| `created_at` | TIMESTAMP | 建立時間 |
| `updated_at` | TIMESTAMP | 更新時間 |

### device_data - 設備數據表

| 欄位 | 類型 | 說明 |
|------|------|------|
| `id` | BIGINT | 主鍵 |
| `device_id` | BIGINT | 設備 ID (FK) |
| `coin_in_count` | INT | 入金計數 |
| `payout_count` | INT | 出金計數 |
| `ball_in_count` | INT | 入球計數 |
| `ball_out_count` | INT | 出球計數 |
| `assign_count` | INT | 開分計數 |
| `settled_count` | INT | 洗分計數 |
| `recorded_at` | TIMESTAMP | 記錄時間 |

> 完整 schema 請參考 migration 檔案

## Redis Key 規範

| Key 格式 | 用途 | TTL |
|----------|------|-----|
| `device:{chip_id}:latest` | 設備最新數據 | 3600s |
| `device:{chip_id}:status` | 設備在線狀態 | 無 |

---
*最後更新：2026-01-06*
