# 變更日誌

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
