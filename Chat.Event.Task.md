## Api -> Server (-> App)

## 功能: officialTaskCreated 官方任務已創建

```bash
request:
  string taskId; 聊天室(任務)ID

  int64 startTime; 任務活動開始時間
  int64 endTime; 任務活動結束時間

  list<{
    string token; 物品token
    string location; 物品GPS位置
    int cooldownTime; 物品CD時間(秒)
    int64 activationTime; 物品啟用時間(時間戳)

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
    }> availableItemContentList;
  }> itemList; 物品配置列表
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息

  case error == 0:
    list<{
      string token; 物品token
      int type; 物品種類. 1: 幣(數量), 2: 拼圖(id), 3: 徽章(數量)
      int value; 物品內容物(根據method所產生的數值)
    }> itemList; 物品配置列表
```

## 功能: taskCreated 任務已創建

```bash
request:
  string taskId; 聊天室(任務)ID
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```

## 功能: taskStarted 任務已開始

> 轉發給任務聊天室所有成員

```bash
request:
  string taskId; 聊天室(任務)ID
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```

## 功能: taskCanceled 任務已取消

> 轉發給任務聊天室所有成員

```bash
request:
  string taskId; 聊天室(任務)ID
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```

## 功能: inviteIntoTask 邀請加入任務

> 轉發給任務聊天室指定成員(userIds)

```bash
request:
  string taskId; 聊天室(任務)ID
  string userIds; 玩家ID列表
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```
