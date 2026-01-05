# Shared Specs - 跨專案共用規範

> **唯一真相來源** - 所有專案必須引用此處規範，禁止自創

## 適用專案

| 專案 | 說明 |
|------|------|
| iot.tg25 | Web VPS (Laravel + Vue) |
| infra | DB VPS (MySQL, MQTT, Redis) |
| vip.tg25.win | 會員系統 |
| esp32-firmware | 設備韌體 |

## 目錄結構

```
shared-specs/
├── mqtt/           # MQTT 主題規範
├── database/       # 資料庫 schema
├── api/            # API 契約
└── naming/         # 命名慣例
```

## 使用方式

### 各專案引用（Git Submodule）
```bash
git submodule add <repo-url> .shared-specs
```

### Kiro Steering 引用
```markdown
#[[file:.shared-specs/mqtt/topics.md]]
```

## 修改規範流程

1. 修改此 repo 的對應文件
2. 提交並推送
3. 各專案執行 `git submodule update --remote`

## 嚴禁行為

- ❌ 在各專案自創規範
- ❌ 複製規範到各專案（會造成版本不同步）
- ❌ 猜測規範內容
