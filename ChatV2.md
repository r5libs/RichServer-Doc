## 功能: verifyUser 驗證 User 登入是否合法

```bash
- request:
  string accessToken 待驗證的 Token
```

```bash
- response:
  int error 0: 成功, -1: Token 錯誤

  case error == 0:
    string userId 玩家ID
    string userName 玩家名稱/暱稱
    uint64 lastOpRevision 最後一筆同步ID(每次sendMessage的opRevision可以理解為lastOpRevision, 初始值為0)
```

## 功能: getJoinedChatList 取得玩家聊天室列表

```bash
- request:
  string userId 玩家ID
```

```bash
- response:
  int error 0: 成功, -1: userId不存在

  case error == 0:
    list<string> chatIdList chatId列表
```

## 功能: getChat 取得聊天室資訊

```bash
- request:
  string chatId 聊天室ID
```

```bash
- response:
  int error 0: 成功, -1: chatId不存在

  case error == 0:
    int chatType 聊天室類型 0: 私聊, 1: 付費群組, 2: 官方群組
    string name 聊天室名稱
    string creatorId 創建者ID
    string lastMessageId 最後一筆訊息ID(每次sendMessage的messageId可以理解為lastMessageId)
    list<string> userIdList userId列表

    case chatType == 1 || chatType == 2:
      string notice 公告訊息
```

## 功能: isChatExist 檢查聊天室 ID 是否可用

```bash
- request:
  string chatId 聊天室ID
```

```bash
- response:
  int error 0: 成功, -1: chatId重複
```

## 功能: createPrivateChat 建立私聊聊天室

> 因為直接建立私聊聊天室, 所以直接把二位玩家的 ID 也一併傳遞

```bash
- request:
  string chatId 聊天室ID
  string member1 1玩家ID
  string member2 2玩家ID
```

```bash
- response:
  int error 0: 成功, -1: chatId重複
```

## 功能: dismissChat 解散聊天室

```bash
- request:
  string chatId 聊天室ID
  string userId 玩家ID
```

```bash
- response:
  int error 0: 成功, -1: chatId不存在, -2: userId非創建者
```

## 功能: joinChat 加入聊天室

```bash
- request:
  string chatId 聊天室ID
  string userId 玩家ID
```

```bash
- response:
  int error 0: 成功, -1: chatId不存在, -2: userId不存在, -3: userId已再聊天室
```

## 功能: leaveChat 離開聊天室

```bash
- request:
  string chatId 聊天室ID
  string userId 玩家ID
```

```bash
- response:
  int error 0: 成功, -1: chatId不存在, -2: userId不存在, -3: userId不再聊天室
```

## 功能: getMessageContentUrl 取得可紀錄訊息內容的網址

```bash
- request:
  string chatId 聊天室ID
  string userId 玩家ID
  string messageId 訊息ID
  int contentType 1: image 影像, 2: video 影片, 3: audio 語音
```

```bash
- response:
  int error 0: 成功
  string contentUrl 網址
```

## 功能: sendMessage 傳送聊天室訊息
> 這裡命名為 send, 但實際上僅是將 user 在此聊天室所做的任何事記錄在 DB\
> 每個玩家須紀錄當前 opRevision 值. 必須可由{userId, opRevision} 查詢 {chatId, messageId, receipt})

```bash
- request:
  string chatId 聊天室ID
  string from 訊息傳送者ID
  string messageId 訊息ID
  binary receipt 該筆訊息要紀錄的資料, 供往後同步使用(Google Protocol Buffer格式)
  // list<{userId, opRevision}> userList 聊天室所有玩家的同步狀態
```

```bash
- response:
  int error 0: 成功, -1: chatId不存在, -2: from不存在, -3: from不再聊天室, -4: messageId重複記錄, -5: from被禁言/封鎖
  
    case error == 0:
      uint64 opRevision from玩家最新的opRevision
```

## 功能: syncMessage 同步聊天室訊息

```bash
- request:
  string userId 玩家ID
  string start 從哪筆訊息開始(opRevision)
  int count 筆數(這裡泛指請求的最大筆數)
```

```bash
- response:
  int error 0: 成功, -1: chatId不存在, -2: start不存在

  case error == 0:
    list<{opRevision, receipt}> messageList
```
