## 如何建立 SocketChannel?

```cs
// 建立SocketChannel
string serverIp = "192.168.1.114";
int serverPort = 26345;
SocketChannel s = await SocketConnector.Connect(serverIp, serverPort);

// 關閉SocketChannel
s.Dispose();
```

## Server Api 命名規範

```bash
[形式1]
Api名稱: xxx.yyy
  xxx: Server名稱
  yyy: Api名稱

範例: login.VerifyUser, chat.SendImage...
```

```bash
[形式2]
Api名稱: zzz.yyy
  zzz: Server類型
  yyy: Api名稱

範例: Chat.SendImage...

Server端必須提供一個RouteMessage函式, 可用來將訊息轉發到指定的chat(Server名稱)
```

### Server Api 訊息結構

```bash
假設Api名稱為login.VerifyUser, 則可能存在下列幾種訊息結構:
  - VerifyUserMessage
  - VerifyUserRequest
  - VerifyUserResponse
  - VerifyUserReceipt

以上訊息結構又可組合成下列幾種形式:
  - VerifyUserMessage 請求不回應的形式
  - VerifyUserRequest, VerifyUserResponse 請求並回應的形式
  - VerifyUserRequest, VerifyUserResponse, VerifyUserReceipt 請求並回應且有回執的形式

具體用法可查看後續的Api介紹
```

## 如何呼叫 Server Api?

```cs
// 請求Api, 不等候回應(Message)

SocketChannel.SendMessage("xxx.yyy", new aaaMessage()
{

});
```

```cs
// 請求Api, 並等候回應(Request/Response)

// [方式1]
aaaResponse response = await SocketChannel.SendRequest<aaaRequest, aaaResponse>("xxx.yyy", new aaaRequest()
{

});

// [方式2]
SocketChannel.SendRequest("xxx.yyy", new aaaRequest()
{

}, (Exception error, aaaResponse response) =>
{

});
```

## 如何接收由 Server 主動通知的訊息?

```cs
SocketChannel.Message += (string messageType, byte[] messagePayload) =>
{
  switch (messageType)
  {
    // 通知訊息傳送失敗(SendText/SendImage等Api執行失敗時)
    case "chat.SendMessageFailed":
      // SendMessageFailedMessage
      break;

    // 通知有新增的聊天室內容
    case "chat.SendReceiptMessage":
      // JoinChatReceipt
      // LeaveChatReceipt
      // KickoutFromChatReceipt
      // SendTextReceipt
      // SendImageReceipt
      // SendImageCheckedReceipt
      // SendMessageReadReceipt
      // RemoveMessageReceipt
      // DismissChatReceipt
      break;

    // 通知聊天室成員已設定禁言
    case "chat.MuteChatMessage":
      // MuteChatMessage
      break;

    // 通知聊天室成員已取消禁言
    case "chat.UnmuteChatMessage":
      // UnmuteChatMessage
      break;

    // 任務狀態異動通知
    case "chat.UpdateTaskStatus":
      // UpdateTaskStatusMessage
      break;

    // 更新玩家在任務中的活動數據
    case "chat.UpdateUserTaskData":
      // UpdateUserTaskDataMessage
      break;
  }
};
```

## 如何取得 MessageResponseException 的 ErrorCode

```cs
// 若Server Api執行異常, 則丟出MessageResponseException例外, 以下為提供的二種方式:

// [方式1]
try
{
  aaaResponse response = await SocketChannel.SendRequest<aaaRequest, aaaResponse>("xxx.yyy", new aaaRequest()
  {

  });
}
catch(Exception e)
{
  if (e is AggregateException)
  {
    e = e.InnerException;
  }
  if (e is MessageResponseException e1)
  {
    // e1.ErrorCode 錯誤碼
  }
  // e.Message 例外訊息
  // e.StackTrace 發生例外時的CallStack(Server Side), 方便排除問題
}

// [方式2]
SocketChannel.SendRequest("xxx.yyy", new aaaRequest()
{

}, (Exception error, aaaResponse response) =>
{
  if (error != null)
  {
    if (e is AggregateException)
    {
      e = e.InnerException;
    }
    if (e is MessageResponseException e1)
    {
      // e1.ErrorCode 錯誤碼
    }
    // e.Message 例外訊息
    // e.StackTrace 發生例外時的CallStack(Server Side), 方便排除問題
  }
  else
  {

  }
});
```

## 關於公頻?

```bash
- 公頻ID可由Chat.OfficialGroupId取得

- 公頻訊息不會經由FetchMessage回傳, 也就是不會被同步. 玩家在線時, 可透過server通知(chat.SendReceiptMessage)取得, 其UserReceiptMessage.OpRevision為0
```

## Api - gate.QueryConnector 取得適合登入的 Connector Server

```bash
QueryConnectorRequest:
```

```bash
QueryConnectorResponse:
  string Name; // Connector Server Name
  string Host; // Connector Server Host
  int32 Port; // Connector Server Port
```

## Api - login.VerifyUser 登入驗證

```bash
VerifyUserRequest:
  string AccessToken; // 待驗證的Token
```

```bash
VerifyUserResponse:
  string UserId; // 玩家ID
  string ChatKey; // 執行chat.UseChat所需要的key
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: Token無效
    -3: 已登入
```

## Api - chat.UseChat 使用聊天服務

```bash
UseChatRequest:
  string ChatKey; // 可經由login.VerifyUser取得
```

```bash
UseChatResponse:
  uint64 LastOpRevision; // 最後一筆訊息紀錄ID
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 尚未登入
    -3: 重複執行chat.UseChat
    -4: ChatKey不存在/比對錯誤
```

## Api - chat.JoinChat 加入聊天室

```bash
JoinChatRequest:
  string ChatId; // 聊天室ID
```

```bash
JoinChatResponse:
  string MessageId; // 訊息ID. 可以用來比對JoinChatReceipt.MessageId
```

```bash
JoinChatReceipt:
  string ChatId; // 聊天室ID
  string MessageId; // 訊息ID
  int64 Timestamp; // 時間戳記
  string MemberId; // 聊天室成員ID(玩家ID)
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 尚未登入
    -3: 無法取得聊天室
    -4: 不能加入私人聊天室
    -5: 聊天室人數已滿
    -6: 已在聊天室
```

## Api - chat.LeaveChat 離開聊天室

```bash
LeaveChatRequest:
  string ChatId; // 聊天室ID
```

```bash
LeaveChatResponse:
  string MessageId; // 訊息ID. 可以用來比對LeaveChatReceipt.MessageId
```

```bash
LeaveChatReceipt:
  string ChatId; // 聊天室ID
  string MessageId; // 訊息ID
  int64 Timestamp; // 時間戳記
  string MemberId; // 聊天室成員ID(玩家ID)
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 尚未登入
    -3: 無法取得聊天室
    -4: 不能離開私人聊天室
    -5: 不在聊天室
```

## Api - chat.KickoutFromChat 剔除聊天室

```bash
KickoutFromChatRequest:
  string ChatId; // 聊天室ID
  string[] MemberIds; // 聊天室成員ID(玩家ID)列表
```

```bash
KickoutFromChatResponse:
  string MessageId; // 訊息ID. 可以用來比對KickoutFromChatReceipt.MessageId
```

```bash
KickoutFromChatReceipt:
  string ChatId; // 聊天室ID
  string MessageId; // 訊息ID
  int64 Timestamp; // 時間戳記
  string[] KickedoutMemberIds; // 已剔除的聊天室成員ID(玩家ID)陣列
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 尚未登入
    -3: 無法取得聊天室
    -4: 此功能只支援群組
    -5: 只有群主才能踢人
    -6: 指定的聊天成員均不在聊天室
```

## Api - chat.SendText 傳送文字訊息

```bash
SendTextRequest:
  string To; // 傳送給誰. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室ID
  string Text; // 文字內容
  string RelatedMessageId; // 回覆訊息ID. 如果是要回覆某筆訊息, 此欄位為該筆的訊息ID, 否則為空字串
```

```bash
SendTextResponse:
  string MessageId; // 訊息ID. 可以用來比對SendTextReceipt.MessageId
```

```bash
SendTextReceipt:
  string From; // 訊息發送者ID
  string To; // 訊息接收者ID. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室ID
  string MessageId; // 訊息ID
  int64 Timestamp; // 時間戳記
  string RelatedMessageId; //回覆訊息ID
  string Text; // 文字內容
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 尚未登入
    -3: 無法取得聊天室
    -4: 訊息傳送者不可為訊息接收者
    -5: 訊息傳送者不在聊天室
    -6: 訊息接收者不在聊天室
```

## Api - chat.SendImage 傳送影像訊息

```bash
SendImageRequest:
  string To; // 傳送給誰. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室ID
  string RelatedMessageId; // 回覆訊息ID. 如果是要回覆某筆訊息, 此欄位為該筆的訊息ID, 否則為空字串
```

```bash
SendImageResponse:
  string MessageId; // 訊息ID. 可以用來比對SendTextReceipt.MessageId
  string ContentUrl; // 上傳影像的Url
```

```bash
SendImageReceipt:
  string From; // 訊息發送者ID
  string To; // 訊息接收者ID. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室ID
  string MessageId; // 訊息ID
  int64 Timestamp; // 時間戳記
  string RelatedMessageId; //回覆訊息ID
  string ContentUrl; // 上傳影像的Url
```

```bash
SendImageCheckedReceipt:
  string From; // 訊息發送者ID
  string To; // 訊息接收者ID. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室ID
  string MessageId; // 訊息ID. 這是影像已上傳完畢所產生的訊息ID
  int64 Timestamp; // 時間戳記
  string OriginalMessageId; // 訊息ID. 這是原本SendImageRequest所產生的訊息ID, 也就是SendImageResponse.MessageId
  string ContentUrl; // 下載影像的Url
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 尚未登入
    -3: 無法取得聊天室
    -4: 訊息傳送者不可為訊息接收者
    -5: 訊息傳送者不在聊天室
    -6: 訊息接收者不在聊天室
```

## Api - chat.FetchMessage 獲取聊天室訊息

> 注意: 在使用 chat.FetchMessage 同步訊息期間, 當聊天室有新增訊息時, chat.SendReceiptMessage 也將會進行通知

```bash
FetchMessageRequest:
  uint64 Start; // 從哪一筆訊息開始(opRevision). server會將此值加1, 也就是從下一筆開始
  int32 Count; // 一次幾筆. 目前最大一次可同步筆數為Chat.FetchingMaxCount
```

```bash
FetchMessageResponse:
  UserReceiptMessage[] Messages; // 取得的訊息陣列
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 玩家尚未登入
    -3: Start大於實際最大值
    -4: Count太大
```

## Api - chat.SendMessageRead 傳送訊息已讀通知

```bash
SendMessageReadRequest:
  string To; // 傳送給誰. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室ID
```

```bash
SendMessageReadResponse:
  string MessageId; // 訊息ID. 可以用來比對SendMessageReadReceipt.MessageId
```

```bash
SendMessageReadReceipt:
  string From; // 訊息發送者ID
  string To; // 訊息接收者ID. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室ID
  string MessageId; // 訊息ID. 這是影像已上傳完畢所產生的訊息ID
  int64 Timestamp; // 時間戳記
  string LastReadMessageId; // 最後一筆已讀訊息ID. 表示在這之前的訊息(包含這筆)都是已讀的
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 尚未登入
    -3: 無法取得聊天室
    -4: 訊息傳送者不可為訊息接收者
    -5: 訊息傳送者不在聊天室
    -6: 訊息接收者不在聊天室
    -7: 已讀狀態只支援私聊
```

## Api - chat.RemoveMessage 移除訊息

```bash
RemoveMessageRequest:
  string ChatId; // 聊天室ID
  string[] MessageIds; // 訊息ID陣列
```

```bash
RemoveMessageResponse:
  string MessageId; // 訊息ID. 可以用來比對RemoveMessageReceipt.MessageId
```

```bash
RemoveMessageReceipt:
  string ChatId; // 聊天室ID
  string MessageId; // 訊息ID
  int64 Timestamp; // 時間戳記
  string[] RemovedMessageIds; // 實際刪除的訊息ID陣列
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 尚未登入
    -3: 無法取得聊天室
    -4: 此功能只支援群組
    -5: 只有群主才能刪除訊息
    -6: 指定的聊天訊息均不在聊天室
```

## Api - chat.DismissChat 解散聊天室

```bash
DismissChatReceipt:
  string ChatId; // 聊天室ID
  string MessageId; // 訊息ID
  int64 Timestamp; // 時間戳記
```

## Api - chat.UpdateTaskData 更新任務活動數據

```bash
UpdateTaskMessage:
  string TaskId; // 聊天室(任務)ID
  string[] DataList;
    // 揪運動: [0]: GPS位置, [1]: 移動量(步數/里程)
```

## chat.SendMessageFailed 通知訊息傳送失敗

```bash
SendMessageFailedMessage:
  string ChatId; // 聊天室ID
  string MessageId; // 訊息ID
```

## chat.SendReceiptMessage 通知有新增的聊天室內容

```bash
ReceiptMessage:
  string MessageType; // 某Api的回執訊息名稱. 一般都是xxx.yyyReceipt的形式
  bytes MessageContent; // 該回執實際紀錄的訊息內容. 對應yyyReceipt的訊息結構, 如SendTextReceipt
```

```bash
UserReceiptMessage:
  uint64 OpRevision; // 該筆回執訊息紀錄ID(App必須持續記錄此值. 下次開啟App時, 使用FetchMessage進行訊息同步)
  bytes ReceiptMessage; // 對應ReceiptMessage結構. 如果長度為0, 表示此筆訊息已被後台刪除, 所以不需要處理, 但OpRevision一樣要記錄
```

## chat.MuteChatMessage 通知聊天室成員已設定禁言

```bash
MuteChatMessage:
  string ChatId; // 聊天室ID
  int64 Timestamp; // 時間戳記(設定禁言時間)
  string MemberId; // 聊天室成員ID(玩家ID)
  int32 TimeLimit; // 禁言的時間(分鐘)
```

## chat.UnmuteChatMessage 通知聊天室成員已取消禁言

```bash
UnmuteChatMessage:
  string ChatId; // 聊天室ID
  int64 Timestamp; // 時間戳記(取消禁言時間)
  string MemberId; // 聊天室成員ID(玩家ID)
```

## chat.UpdateTaskStatus

```bash
TaskStatus:
  CREATED = 0;
  STARTED = 1;
  COMPLETED = 2;
  CANCELED = 3;

UpdateTaskStatusMessage:
  string TaskId; // 聊天室(任務)ID
  TaskStatus Status; // TaskStatus.CREATED / TaskStatus.STARTED / TaskStatus.COMPLETED / TaskStatus.CANCELED
```

## chat.UpdateUserTaskData

```bash
UpdateUserTaskDataMessage:
  string TaskId; // 聊天室(任務)ID
  string UserId; // 玩家ID
  string[] TaskDataList;
    // 揪運動: [0]: 所有玩家總移動量(步數/里程)
  string[] UserDataList;
    // 揪運動: [0]: GPS位置, [1]: 玩家個人總移動量(步數/里程)
```
