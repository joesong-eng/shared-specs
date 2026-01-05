# Shared-Specs 設定指南

## 這是什麼？

`shared-specs` 是跨專案的共用規範倉庫，解決以下問題：
- 規範分散在各專案，改一處漏改其他
- AI 或開發者自創規範，造成不一致
- MQTT 主題、API 契約等沒有統一來源

## 專案架構

```
GitHub (joesong-eng)
    │
    └── shared-specs ← 唯一真相來源
            │
            ├── mqtt/topics.md      # MQTT 主題規範
            ├── api/contracts.md    # API 契約
            ├── database/schema.md  # 資料庫規範
            └── naming/conventions.md

引用此規範的專案：
┌─────────────────┐
│ iot.tg25        │ ← Web VPS (Laravel + Vue)
├─────────────────┤
│ infra           │ ← DB VPS (MySQL, MQTT, Redis, listener.py)
├─────────────────┤
│ vip.tg25.win    │ ← 會員系統
├─────────────────┤
│ esp32-firmware  │ ← 設備韌體 (iotWork_V3.0)
└─────────────────┘
```

## 使用規則

### ✅ 正確做法
- 修改規範 → 改 `shared-specs` repo → 各專案 pull
- 引用規範 → 讀 `.shared-specs/` 目錄
- AI 開發時 → 必須先讀取 shared-specs 再實作

### ❌ 禁止行為
- 在各專案自創 MQTT 主題、API 格式
- 複製規範內容到各專案（會造成版本不同步）
- 猜測規範內容
- AI 自行決定欄位名稱或格式

---

## 各專案待辦清單

### ESP32 韌體 (iotWork_V3.0) ✅ 已完成
- [x] 加入 shared-specs（直接 clone）
- [x] MQTT 主題符合規範
- [x] 訊息格式符合規範

### iot.tg25 (Web VPS) ✅ 已完成
- [x] 加入 shared-specs submodule
- [x] 更新 steering 引用共用規範

### tg25-infra (DB VPS) ⚠️ 待處理
- [ ] 加入 shared-specs submodule
- [ ] 更新 listener.py 訂閱新主題
  - `device/+/config/timezone`
  - `device/+/events/daily_reset`
- [ ] 實作欄位名稱轉換（韌體 → 後端）
- [ ] 更新 steering 引用共用規範

### vip.tg25.win ⚠️ 待處理
- [ ] 加入 shared-specs submodule
- [ ] 更新 steering 引用共用規範

---

## 設定步驟

### 1. 各專案加入 submodule

```bash
# 在專案根目錄執行
git submodule add git@github.com:joesong-eng/shared-specs.git .shared-specs
git commit -m "feat: 加入共用規範 submodule"
git push
```

### 2. Clone 專案時記得拉 submodule

```bash
git clone --recurse-submodules git@github.com:joesong-eng/<project>.git

# 或已經 clone 的專案
git submodule update --init
```

### 3. 更新規範

```bash
# 在任一專案中更新 shared-specs
cd .shared-specs
git pull origin main
cd ..
git add .shared-specs
git commit -m "chore: 更新共用規範"
```

---

## 聯絡

- GitHub: joesong-eng
- 網站: tg.tg25.win
