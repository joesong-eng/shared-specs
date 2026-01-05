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
| `mqtt_topic` | VARCHAR(100) | MQTT 主題 |
| `last_heartbeat` | TIMESTAMP | 最後心跳時間 |

> 完整 schema 請參考 migration 檔案

## Redis Key 規範

| Key 格式 | 用途 | TTL |
|----------|------|-----|
| `device:{device_uuid}:latest` | 設備最新數據 | 3600s |

---
*最後更新：2026-01-05*
