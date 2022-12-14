## Server -> Api

## 功能: getAllOfficialTask 取得所有官方任務列表

```bash
request:
```

```bash
response:
  int error; 0: 成功
  list<string> taskModes; 任務模式列表
```

## 功能: getOfficialTask 取得官方任務

```bash
request:
  string taskMode; 任務模式: 1: 海線美食大冒險
```

```bash
response:
  int error; 0: 成功, -1: task不存在

  case error == 0:
    int status: 任務狀態. 0: 尚未開始, 1: 進行中, 2: 已結束(失敗), 3: 已結束(成功), 4: 已取消

    int64 startTime; 任務活動開始時間
    int64 endTime; 任務活動結束時間

    list<{
      string token; 物品token
      string location; 物品GPS位置
      int cooldownTime; 物品CD時間(秒)
      int64 activationTime; 物品可被拿取時間(時間戳). 此值分二種情況, 1: 指任務開始後, 首次出現的時間. 2: 被拿取後, CD結束後的時間

      int contentType; 物品種類. 0: 尚未設定, 1: 幣(數量), 2: 拼圖(id), 3: 徽章(數量)
      int contentValue; 物品內容物. (如果contentType為0, 此值被忽略, 填0即可)

      list<{
        int type; 物品種類. 1: 幣(數量), 2: 拼圖(id), 3: 徽章(數量)
        int method; 物品設定. 1: 範圍取值, 2: 機率取值
        int values[];
          case method == 1:
            values[0]: 最小值
            values[1]: 最大值
          case method == 2:
            values[0]: 數值1
            values[1]: 數值1出現機率
            values[2]: 數值2
            values[3]: 數值2出現機率
      }> settingsList; 計算物品內容物可選的方式
    }> itemList; 物品配置列表
```

## 功能: takeOfficialTaskItem 通知玩家取得官方任務物品

```bash
request:
  string taskMode; 任務模式
  string userId; 那位玩家ID取得
  string itemToken; 取得的物品Token
```

```bash
response:
  int error; 0: 成功, -1: task不存在, -2: task尚未開始, -3: task已結束, -4: 物品token無效, -5: 物品不可取(如CD期間)
```

## 功能: updateOfficialTaskItems 更新官方任務物品

```bash
request:
  string taskMode; 任務模式

  list<{
    string token; 物品token
    int64 activationTime; 物品下次可被拿取的時間(時間戳). 這裡被計算為CD結束後的時間, 但若是首次設定, 則直接回傳getOfficialTask所取得的時間
    int contentType; 物品種類. 1: 幣(數量), 2: 拼圖(id), 3: 徽章(數量)
    int contentValue; 物品內容物(根據method所產生的數值)
  }> itemList; 物品配置列表
```

```bash
response:
  int error; 0: 成功, -1: task不存在, -2: task尚未開始, -3: task已結束

  case error == 0:
    list<{
      string token; 物品原token
      string refreshToken; 物品更新後的token
    }> itemList; 更新後的物品列表
```

## 功能: updateOfficialTaskStatus 更新官方任務狀態

```bash
request:
  string taskMode; 任務模式
  int status: 任務狀態. 2: 已結束(失敗), 3: 已結束(成功), 4: 已取消
```

```bash
response:
  int error; 0: 成功, -1: task不存在, -2: task尚未開始, -3: task已結束
```

## 功能: queryOfficialTaskMode 經由聊天室 ID 查詢官方任務模式

```bash
request:
  string chatId; 聊天室ID
```

```bash
response:
  int error; 0: 成功, -1: chatId不存在, -2: chatId不屬於官方任務(團)

  case error == 0:
    int taskMode; 任務模式
```
