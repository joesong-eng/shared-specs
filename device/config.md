# 設備配置規範

> **唯一真相來源** - ESP32 設備配置結構

## 配置結構

### WiFi 配置

| 欄位 | 類型 | 說明 | 預設值 |
|------|------|------|--------|
| `ssid` | string | WiFi SSID | - |
| `password` | string | WiFi 密碼 | - |
| `autoConnect` | bool | 自動連線 | false |
| `timeout` | uint32 | 連線超時 (ms) | 10000 |

### MQTT 配置

| 欄位 | 類型 | 說明 | 預設值 |
|------|------|------|--------|
| `mqtt_server` | string | MQTT 伺服器 | `direct-mqtt.tg25.win` |
| `port` | uint16 | 連接埠 | 8883 (TLS) |
| `client_id` | string | 客戶端 ID | - |
| `username` | string | 使用者名稱 | - |
| `password` | string | 密碼 | - |
| `use_tls` | bool | 使用 TLS | true |
| `keepalive` | uint32 | 心跳間隔 (秒) | 60 |
| `timeout` | uint32 | 連線超時 (ms) | 5000 |

### 設備憑證

| 欄位 | 類型 | 說明 |
|------|------|------|
| `chip_id` | string | 設備 ID（用於 MQTT 主題） |
| `auth_key` | string | 認證金鑰 |
| `device_type` | string | 設備類型 |
| `firmware_version` | string | 韌體版本 |

### 時區配置

| 欄位 | 類型 | 說明 | 預設值 |
|------|------|------|--------|
| `timezone_id` | string | IANA 時區 ID | `Asia/Taipei` |
| `display_name` | string | 顯示名稱 | `台灣標準時間` |
| `offset_hours` | int | 時區偏移小時 (-12~+14) | 8 |
| `offset_minutes` | int | 時區偏移分鐘 (0,15,30,45) | 0 |
| `auto_dst` | bool | 自動夏令時 | false |
| `ntp_server` | string | 主要 NTP 伺服器 | `pool.ntp.org` |
| `backup_ntp` | string | 備用 NTP 伺服器 | `time.windows.com` |

## AP 模式配置

設備進入 AP 模式時的預設值：

| 項目 | 值 |
|------|-----|
| SSID | `WawAP` |
| 密碼 | `qq123123` |
| IP | `192.168.22.22` |
| 閘道 | `192.168.22.22` |
| 子網路 | `255.255.255.0` |
| 超時 | 10 分鐘 |

## 繼電器配置 (Remote Command System)

| 欄位 | 類型 | 說明 | 預設值 |
|------|------|------|--------|
| `assign_credit_pin` | uint8 | 開分 GPIO 腳位 | 4 |
| `settle_credit_pin` | uint8 | 洗分 GPIO 腳位 | 5 |
| `pulse_duration_ms` | uint16 | 脈衝時長 (ms) | 100 |
| `pulse_interval_ms` | uint16 | 脈衝間隔 (ms) | 200 |
| `active_low` | bool | 低電平有效 | true |

**說明：**
- 繼電器使用 active-low 邏輯（0V = 觸發）
- 開分/洗分指令透過 GPIO 脈衝控制外部繼電器
- 多次開分時，每次脈衝間隔 `pulse_interval_ms`

## 系統常數

| 常數 | 值 | 說明 |
|------|-----|------|
| `MQTT_QOS_LEVEL` | 1 | MQTT QoS 等級 |
| `MQTT_BUFFER_SIZE` | 1024 | 訊息緩衝區大小 |
| `MQTT_RETRY_MAX_COUNT` | 3 | 重連最大次數 |
| `MQTT_RETRY_INTERVAL_MS` | 5000 | 重連間隔 (ms) |
| `DEBOUNCE_DELAY` | 50 | 按鈕去抖延遲 (ms) |
| `LONG_PRESS_THRESHOLD` | 2000 | 長按閾值 (ms) |

---
*來源：iotWork_V3.0/src/config/*
*最後更新：2026-01-06*
