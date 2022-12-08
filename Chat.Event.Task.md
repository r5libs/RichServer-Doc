## Api -> Server (-> App)

## 功能: createTask 創建任務

```bash
request:
  string taskId; 聊天室(任務)ID
  string taskName; 聊天室(任務)名稱
  string taskCreator; 聊天室(任務)創建人
  int taskType; 任務類型. 1: 揪運動
  int taskMode; 任務模式. 揪運動: 1: 步數模式, 2: 里程模式
  string[] invitingUsers; 任務邀請列表(玩家ID)
  int[] meetConditions; 任務達成條件列表. 揪運動: 0: 步數/里程目標
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```

## 功能: startTask 開始任務

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

## 功能: cancelTask 取消任務

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

> 轉發給任務聊天室指定成員(userId)

```bash
request:
  string taskId; 聊天室(任務)ID
  string userId; 玩家ID
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```
