# MQTT 實時監控頁面整合指南

> 給 WAWv7 和 WAWv8 專案的前端實時監控整合說明

## 連線資訊

| 項目 | 值 |
|------|-----|
| WebSocket URL | `wss://mqtt.tg25.win/mqtt-ws/` |
| 用戶名 | `waw_user` |
| 密碼 | `waw_mqtt_2024` |

## 訂閱主題

依據 `topics.md` 規範：

```javascript
// 數據更新 (設備每次計數變化時發送)
client.subscribe('device/+/data/update');

// 設備狀態 (上線/離線)
client.subscribe('device/+/status');
```

## 訊息格式

### 數據更新 (`device/{chip_id}/data/update`)

```json
{
  "credit_in": 100,      // 入金計數
  "credit_out": 50,      // 出金計數
  "ball_in": 0,          // 入球計數
  "ball_out": 0,         // 出球計數
  "assign_credit": 10,   // 開分計數
  "settled_credit": 5,   // 洗分計數
  "last_activity": 1704499200
}
```

### 設備狀態 (`device/{chip_id}/status`)

純文字：`online` 或 `offline`

## 前端整合範例

### 方案 1：純 JavaScript (推薦用於簡單頁面)

```html
<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
<script>
const client = mqtt.connect('wss://mqtt.tg25.win/mqtt-ws/', {
    username: 'waw_user',
    password: 'waw_mqtt_2024',
    reconnectPeriod: 3000,
});

client.on('connect', () => {
    console.log('MQTT 已連線');
    client.subscribe('device/+/data/update');
    client.subscribe('device/+/status');
});

client.on('message', (topic, message) => {
    const parts = topic.split('/');
    const chipId = parts[1];
    
    if (topic.includes('/data/update')) {
        const data = JSON.parse(message.toString());
        console.log(`${chipId}: 入金=${data.credit_in}, 出金=${data.credit_out}`);
        // 更新 UI...
    } else if (topic.includes('/status')) {
        const status = message.toString();
        console.log(`${chipId}: ${status}`);
        // 更新狀態指示燈...
    }
});
</script>
```

### 方案 2：Vue 3 Composable

```typescript
// composables/useMqtt.ts
import mqtt from 'mqtt';
import { ref, onMounted, onUnmounted } from 'vue';

interface DeviceData {
  chipId: string;
  creditIn: number;
  creditOut: number;
  online: boolean;
  lastUpdate: Date;
}

export function useMqtt() {
  const devices = ref<Map<string, DeviceData>>(new Map());
  const connected = ref(false);
  let client: mqtt.MqttClient | null = null;

  onMounted(() => {
    client = mqtt.connect('wss://mqtt.tg25.win/mqtt-ws/', {
      username: 'waw_user',
      password: 'waw_mqtt_2024',
      reconnectPeriod: 3000,
    });

    client.on('connect', () => {
      connected.value = true;
      client?.subscribe('device/+/data/update');
      client?.subscribe('device/+/status');
    });

    client.on('close', () => {
      connected.value = false;
    });

    client.on('message', (topic, message) => {
      const parts = topic.split('/');
      const chipId = parts[1];
      
      if (topic.includes('/data/update')) {
        const data = JSON.parse(message.toString());
        devices.value.set(chipId, {
          chipId,
          creditIn: data.credit_in,
          creditOut: data.credit_out,
          online: true,
          lastUpdate: new Date(),
        });
      } else if (topic.includes('/status')) {
        const status = message.toString();
        const existing = devices.value.get(chipId) || {
          chipId,
          creditIn: 0,
          creditOut: 0,
          lastUpdate: new Date(),
        };
        devices.value.set(chipId, {
          ...existing,
          online: status === 'online',
        });
      }
    });
  });

  onUnmounted(() => {
    client?.end();
  });

  return { devices, connected };
}
```

使用方式：

```vue
<template>
  <div>
    <span :class="connected ? 'online' : 'offline'">
      {{ connected ? '已連線' : '未連線' }}
    </span>
    <table>
      <tr v-for="[id, device] in devices" :key="id">
        <td>{{ device.chipId }}</td>
        <td>{{ device.creditIn }}</td>
        <td>{{ device.creditOut }}</td>
        <td>{{ device.online ? '在線' : '離線' }}</td>
      </tr>
    </table>
  </div>
</template>

<script setup>
import { useMqtt } from '@/composables/useMqtt';
const { devices, connected } = useMqtt();
</script>
```

### 方案 3：Laravel Livewire + Echo

如果後端已經有 Laravel Echo 設定，可以讓 listener 透過 Redis 廣播：

```php
// 在 Laravel 中監聽 Redis 頻道
// listener.py 已經會寫入 Redis: device:{chip_id}:latest

// Livewire 組件
class DeviceMonitor extends Component
{
    public $devices = [];

    public function getListeners()
    {
        return [
            'echo:devices,DeviceUpdated' => 'handleUpdate',
        ];
    }

    public function handleUpdate($data)
    {
        $this->devices[$data['chip_id']] = $data;
    }
}
```

## 欄位對照表

| MQTT 欄位 | 說明 | 後端 DB 欄位 |
|-----------|------|--------------|
| `credit_in` | 入金計數 | `coin_in_count` |
| `credit_out` | 出金計數 | `payout_count` |
| `ball_in` | 入球計數 | `ball_in_count` |
| `ball_out` | 出球計數 | `ball_out_count` |
| `assign_credit` | 開分計數 | `assign_count` |
| `settled_credit` | 洗分計數 | `settled_count` |

## 注意事項

1. **只能讀取，不能發布** - `waw_user` 帳號只有訂閱權限
2. **自動重連** - 設定 `reconnectPeriod: 3000` 確保斷線後自動重連
3. **離線判斷** - 如果 30 秒沒收到數據，可視為離線
4. **chip_id** - 主題中的 `{chip_id}` 對應資料庫的 `device_uuid`

## 參考實作

完整範例：`tg25-infra/web/mqtt-monitor.html`

線上預覽：https://mqtt.tg25.win/

---
*最後更新：2026-01-06*
