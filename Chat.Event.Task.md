## Api -> Server (-> App)

## 功能: taskCreated 任務已創建

```bash
request:
  string taskId; 聊天室(任務)ID
  int64 expirationDate; 任務失效日期(timestamp格式)
  int autoDismissSeconds; 任務結束後的解散時間(秒數). 0: 不解散
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
