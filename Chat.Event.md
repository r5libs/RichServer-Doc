## DB 通知 Server

```bash
Method: POST
Url: http://192.168.1.114:17558/
```

## 功能: messageContentUploaded 通知玩家已上傳訊息內容

```bash
request:
  string chatId; 聊天室ID
  string userId; 玩家ID
  string messageId; 訊息ID
  string messageContentUrl; 可供下載的網址
  int messageContentType; 1: image 影像, 2: video 影片, 3: audio 語音
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```

```bash
example:
curl -X POST -H 'Content-Type: application/x-www-form-urlencoded' -d 'data={"type":"messageContentUploaded","payload":{"chatId":"1111","userId":"2222","messageId":"3333","messageContentUrl":"http://1234/","messageContentType":1}}' http://192.168.1.114:17558/
```

## 功能: joinChat 通知玩家已加入聊天群組

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

```bash
example:
curl -X POST -H 'Content-Type: application/x-www-form-urlencoded' -d 'data={"type":"joinChat","payload":{"chatId":"1111","userId":"2222"}}' http://192.168.1.114:17558/
```

## 功能: leaveChat 通知玩家已離開聊天群組

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

```bash
example:
curl -X POST -H 'Content-Type: application/x-www-form-urlencoded' -d 'data={"type":"leaveChat","payload":{"chatId":"1111","userId":"2222"}}' http://192.168.1.114:17558/
```

## 功能: dismissChat 通知玩家解散聊天群組

```bash
request:
  string[] chatIds; 聊天室ID
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```

```bash
example:
curl -X POST -H 'Content-Type: application/x-www-form-urlencoded' -d 'data={"type":"dismissChat","payload":{"chatIds":["1111","2222"]}' http://192.168.1.114:17558/
```

## 功能: muteChat 通知玩家禁言

```bash
request:
  string chatId; 聊天室ID
  string userId; 玩家ID
  int timeLimit; 0: 沒禁言, >0: 被禁言的時間(分鐘)
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```

```bash
example:
curl -X POST -H 'Content-Type: application/x-www-form-urlencoded' -d 'data={"type":"muteChat","payload":{"chatId":"1111","userId":"2222","timeLimit":0}}' http://192.168.1.114:17558/
```
