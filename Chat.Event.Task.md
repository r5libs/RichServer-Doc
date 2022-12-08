## 功能: startTask 開始任務

```bash
request:
  string chatId; 聊天室ID
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```

## 功能: cancelTask 取消任務

```bash
request:
  string chatId; 聊天室ID
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```

## 功能: inviteIntoTask 邀請加入任務

```bash
request:
  string chatId; 聊天室ID
  string userId; 玩家ID
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```
