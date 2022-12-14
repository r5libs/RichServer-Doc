## OpRevision 欄位解說

```bash
每個玩家都會有個 LastOpRevision 欄位. 此值是個累加的數值, 代表著當前最新的 OpRevision
每個 OpRevision 數值會對應到某個聊天室的某個 MessageId

每個 MessageId 也會對應到一個 Receipt. 這是個 Binary Array, 記錄著往後同步所需要的資料
不同玩家的 OpRevision 通常會對應到相同 MessageId

以下為JS示意程式碼
```

```js
const chats = {
  1000: {
    messageIds: {
      1: {
        receipt: [1, 2, 3, 4, 5],
      },
    },
  },
};

const users = {
  UserA: {
    opRevisions: {
      345: {
        chatId: "1000",
        messageId: "1",
      },
    },
  },
  UserB: {
    opRevisions: {
      1125: {
        chatId: "1000",
        messageId: "1",
      },
    },
  },
};

// 玩家A(opRevision=345) 與 玩家B(opRevision=1125) 對應到相同聊天室(chatId=1000)的訊息ID(messageId=1)

{
  const { chatId, messageId } = users["UserA"].opRevisions[345];
  const { receipt } = chats[chatId].messageIds[messageId];
}

{
  const { chatId, messageId } = users["UserB"].opRevisions[1125];
  const { receipt } = chats[chatId].messageIds[messageId];
}
```

## 功能: verifyUser 驗證 User 登入是否合法

```bash
request:
  string accessToken; 待驗證的 Token
```

```bash
response:
  int error; 0: 成功, -1: Token 錯誤

  case error == 0:
    string userId; 玩家ID
    string userName; 玩家名稱/暱稱
    uint64 lastOpRevision; 當前最新的opRevision數值(每次sendMessage的opRevision可以理解為lastOpRevision, 初始值為0)
```

## 功能: isUserExist 是否為已存在的玩家 ID

```bash
request:
  string userId; 玩家ID
```

```bash
response:
  int error; 0: 成功, -1: userId不存在
```

## 功能: getJoinedChatList 取得玩家聊天室列表

```bash
request:
  string userId; 玩家ID
```

```bash
response:
  int error; 0: 成功, -1: userId不存在

  case error == 0:
    list<string> chatIdList; chatId列表
```

## 功能: getLastOpRevision

```bash
request:
  list<string> userIdList; 玩家ID列表
```

```bash
response:
  int error; 0: 成功

  case error == 0:
    list<{string userId, uint64 lastOpRevision}> userList; 每個userId所對應的opRevision數值
```

## 功能: getChat 取得聊天室資訊

```bash
request:
  string chatId; 聊天室ID
```

```bash
response:
  int error; 0: 成功, -1: chatId不存在

  case error == 0:
    int type; 聊天室類型 0: 私聊, 1: 付費群組, 2: 官方群組(目前ID為00000000000000000000000000000000), 3: 任務群組, 4: 官方任務群組
    string name; 聊天室名稱
    string creatorId; 創建者ID
    string lastMessageId; 當前最新的訊息ID(每次sendMessage的messageId可以理解為lastMessageId)
    list<string> userIdList; 玩家ID列表

    case type == 1 || type == 2:
      string notice; 公告訊息
```

## 功能: isChatExist 檢查聊天室 ID 是否可用

```bash
request:
  string chatId; 聊天室ID
```

```bash
response:
  int error; 0: 成功, -1: chatId重複
```

## 功能: createPrivateChat 建立私聊聊天室

> 因為直接建立私聊聊天室, 所以直接把二位玩家的 ID 也一併傳遞

```bash
request:
  string chatId; 聊天室ID
  string member1; 1玩家ID
  string member2; 2玩家ID
```

```bash
response:
  int error; 0: 成功, -1: chatId重複, -2: member1不存在/member2不存在
```

## 功能: joinChat 加入聊天室

```bash
request:
  string chatId; 聊天室ID
  string userId; 玩家ID
```

```bash
response:
  int error; 0: 成功, -1: chatId不存在, -2: userId不存在/已再聊天室
```

## 功能: leaveChat 離開聊天室

```bash
request:
  string chatId; 聊天室ID
  string userId; 玩家ID
```

```bash
response:
  int error; 0: 成功, -1: chatId不存在, -2: userId不存在/不再聊天室
```

## 功能: getMessageContentUrl 取得可紀錄訊息內容的網址

```bash
request:
  string chatId; 聊天室ID
  string userId; 玩家ID
  string messageId; 訊息ID
  int contentType; 1: image 影像, 2: video 影片, 3: audio 語音
```

```bash
response:
  int error; 0: 成功, -1: chatId不存在, -2: userId不存在/不再聊天室, -3: messageId重複, -4: contentType錯誤, -5: userId被禁言/封鎖

  case error == 0:
    string contentUrl; 網址
```

## 功能: sendMessage 傳送聊天室訊息

> 這裡命名為 send, 但實際上僅是將 user 在此聊天室所做的任何事記錄在 DB

```bash
request:
  string chatId; 聊天室ID
  string messageId; 訊息ID
  string from; 訊息傳送者ID(玩家ID)
  binary receipt; 該筆訊息要紀錄的資料, 供往後同步使用(Google Protocol Buffer格式)
  list<{string userId, uint64 opRevision}> userList; 該聊天室所有玩家的opRevision數值
```

```bash
response:
  int error; 0: 成功, -1: chatId不存在, -2: from不存在/不再聊天室(不判斷), -3: from被禁言/封鎖(不判斷), -4: messageId重複
```

## 功能: fetchMessage 獲取聊天室訊息

```bash
request:
  string userId; 玩家ID
  uint64 start; 從哪筆訊息開始(opRevision)
  int count; 筆數(這裡泛指請求的最大筆數, 實際回傳可能小於此值)
```

```bash
response:
  int error; 0: 成功, -1: userId不存在, -2: start不存在

  case error == 0:
    list<{uint64 opRevision, binary receipt}> messageList; userId每個opRevision所對應的receipt資料
```

## 功能: removeMessage 刪除聊天室訊息

```bash
request:
  string chatId; 聊天室ID
  list<string> messageIdList; messageId列表
```

```bash
response:
  int error; 0: 成功, -1: chatId不存在

  case error == 0:
    list<string> invalidMessageIdList; 無效的筆數(messageId不在聊天室). 這裡假設資料都是正確的, 所以只回傳有誤的
```

## 功能: kickoutFromChat 將玩家從聊天室踢除

```bash
request:
  string chatId; 聊天室ID
  list<string> userIdList; 玩家ID列表
```

```bash
response:
  int error; 0: 成功, -1: chatId不存在

  case error == 0:
    list<string> invalidUserIdList; 無效的筆數(userId不在聊天室). 這裡假設資料都是正確的, 所以只回傳有誤的
```

## 功能: dismissChat 解散聊天室

```bash
request:
  string chatId; 聊天室ID
```

```bash
response:
  int error; 0: 成功, -1: chatId不存在
```

## 功能: isUserMuted 檢查玩家是否被禁言

```bash
request:
  string chatId; 聊天室ID
  string userId; 玩家ID
```

```bash
response:
  int error; 0: 沒被禁言, 1: 禁言中, -1: chatId不存在, -2: userId不存在/不再聊天室
```
