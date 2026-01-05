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
    └── shared-specs ← 你現在在這裡
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
│ esp32-firmware  │ ← 設備韌體
└─────────────────┘
```

## 設定步驟

### 1. 建立 shared-specs repo（只做一次）

```bash
cd shared-specs
git init
git add .
git commit -m "init: 跨專案共用規範"
git remote add origin git@github.com:joesong-eng/shared-specs.git
git push -u origin main
```

### 2. 各專案加入 submodule

在每個專案的根目錄執行：

```bash
# iot.tg25
git submodule add git@github.com:joesong-eng/shared-specs.git .shared-specs
git commit -m "feat: 加入共用規範 submodule"
git push

# infra、vip.tg25.win、esp32-firmware 同樣操作
```

### 3. Clone 專案時記得拉 submodule

```bash
git clone --recurse-submodules git@github.com:joesong-eng/iot.tg25.git

# 或已經 clone 的專案
git submodule update --init
```

### 4. 更新規範

```bash
# 在任一專案中更新 shared-specs
cd .shared-specs
git pull origin main
cd ..
git add .shared-specs
git commit -m "chore: 更新共用規範"
```

## 使用規則

### ✅ 正確做法
- 修改規範 → 改 `shared-specs` repo → 各專案 pull
- 引用規範 → 讀 `.shared-specs/` 目錄

### ❌ 禁止行為
- 在各專案自創 MQTT 主題、API 格式
- 複製規範內容到各專案（會造成版本不同步）
- 猜測規範內容

## 各專案待辦清單

### iot.tg25 (Web VPS) ✅ 已完成
- [x] 加入 shared-specs submodule
- [x] 更新 steering 引用共用規範

### tg25-infra (DB VPS)
- [ ] 加入 shared-specs submodule
- [ ] 更新 listener.py 訂閱新主題
- [ ] 更新 steering 引用共用規範

### ESP32 韌體 (iotWork_V3.0)
- [ ] 加入 shared-specs submodule（如果有 git）
- [ ] 確認主題與規範一致

### vip.tg25.win
- [ ] 加入 shared-specs submodule
- [ ] 更新 steering 引用共用規範

---

## 聯絡

- GitHub: joesong-eng
- 網站: tg.tg25.win
