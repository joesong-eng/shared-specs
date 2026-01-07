# 變更日誌

## 2026-01-08

### 新增：擺錘警報功能

- 新增 MQTT 主題 `device/{chip_id}/events/alarm` - 擺錘警報通知
- 警報類型：`tilt`（機台傾斜）
- 狀態：`triggered`（觸發）/ `cleared`（解除）

### 移除：ball_in / ball_out 欄位

- 從 `data/update` 訊息格式中移除 `ball_in`、`ball_out`、`bill_denomination` 欄位
- 金蘋果彈珠台不需要入球/出球計數功能
- 簡化數據結構，只保留入金/出金相關欄位

**影響範圍：**
| 專案 | 需要修改 | 說明 |
|------|----------|------|
| ESP32 韌體 | ⚠️ 待實作 | 移除 ball_in/ball_out，新增警報功能 |
| infra | ⚠️ 待處理 | listener.py 移除相關欄位對照，訂閱警報主題 |
| iot.tg25 | ⚠️ 待處理 | 資料庫 schema 可保留欄位但不再使用 |

## 2026-01-06

### 新增規範文件

- `device/config.md` - 設備配置規範（WiFi、MQTT、時區等）
- 更新 `api/contracts.md` - 補充完整欄位和認證回應格式
- 更新 `database/schema.md` - 補充完整表格欄位
- 更新 `naming/conventions.md` - 補充 MQTT 主題、時區 ID 命名

## 2026-01-05

### ⚠️ 重大變更：MQTT 欄位名稱統一

**變更內容：**
- `coin_out` → `credit_out`

**原因：**
- 原本 `credit_in` 和 `coin_out` 命名不一致
- 統一使用 `credit_in` / `credit_out`

**影響範圍：**

| 專案 | 需要修改 | 說明 |
|------|----------|------|
| ESP32 韌體 | ✅ 已完成 | MQTTModule.cpp 已更新 |
| infra | ⚠️ 待處理 | listener.py 欄位對照需更新 |
| iot.tg25 | 檢查 | 如有直接讀 MQTT 欄位需更新 |

**部署順序：**
1. **先** 更新 `infra/listener.py` - 支援新欄位 `credit_out`
2. **後** 部署 ESP32 新韌體

**listener.py 修改範例：**
```python
# 舊的
FIELD_MAPPING = {
    "credit_in": "coin_in_count",
    "coin_out": "payout_count",  # ← 舊
    ...
}

# 新的
FIELD_MAPPING = {
    "credit_in": "coin_in_count",
    "credit_out": "payout_count",  # ← 改這裡
    ...
}
```

---
*如有問題請聯繫 @joesong-eng*
