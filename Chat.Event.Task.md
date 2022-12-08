## 功能: startTask 開始任務

> 轉發給任務聊天室所有成員

```bash
request:
  string taskId; 聊天室(任務)ID
  int taskType; 1: 揪運動
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```

## 功能: cancelTask 取消任務

> 轉發給任務聊天室所有成員

```bash
request:
  string taskId; 聊天室(任務)ID
  int taskType; 1: 揪運動
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```

## 功能: inviteIntoTask 邀請加入任務

> 轉發給任務聊天室指定成員(userId)

```bash
request:
  string taskId; 聊天室(任務)ID
  int taskType; 1: 揪運動
  string userId; 玩家ID
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```
