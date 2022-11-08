## 如何建立 SocketChannel

```bash
  string serverIp = "192.168.1.114";
  int serverPort = 26345;
  SocketChannel s = await SocketConnector.Connect(serverIp, serverPort);
```

## 如何呼叫 Server Api?

```bash
- 完整的Api名稱(xxx.yyy)
  xxx: server名稱
  yyy: Api名稱

  範例: login.VerifyUser, chat.SendImage 等

- 請求Api, 不等候回應(Message)
  SocketChannel.SendMessage("xxx.yyy", new aaaMessage()
  {

  });

- 請求Api, 並等候回應(Request/Response)
  [方式1]
  aaaResponse response = await SocketChannel.SendRequest<aaaRequest, aaaResponse>("xxx.yyy", new aaaRequest()
  {

  });

  [方式2]
  SocketChannel.SendRequest("xxx.yyy", new aaaRequest()
  {

  }, (Exception error, aaaResponse response) =>
  {
    if (error != null)
    {

    }
    else
    {

    }
  });
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

  具體用法可察看後續的Api介紹
```

## 如何接收由 Server 主動通知的訊息?

```bash
SocketChannel.Message += (string messageType, byte[] messagePayload) =>
{
  switch (messageType)
  {
    // 通知訊息傳送失敗(SendText/SendImage等Api執行失敗時)
    case "chat.SendMessageFailed":
      break;

    // 通知有新增的聊天室內容
    case "chat.SendReceiptMessage":
      break;
  }
};
```

## 如何取得 MessageResponseException 的 ErrorCode

```bash
若Server Api執行異常, 則丟出MessageResponseException例外, 以下為提供的二種方式:

[方式1]
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

[方式2]
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
    -2: 玩家Token無效
    -3: 玩家已登入
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
    -2: 玩家尚未登入
    -3: 玩家重複執行chat.UseChat
    -4: 玩家ChatKey不存在
    -5: 玩家ChatKey比對錯誤
    -16384: 內部錯誤
```

## Api - chat.JoinChat 加入聊天室

```bash
JoinChatRequest:
  string ChatId; // 聊天室ID
```

```bash
JoinChatResponse:
```

```bash
JoinChatReceipt:
  string ChatId; // 聊天室ID
  string MemberId; // 聊天室成員ID(玩家ID)
  string MessageId; // 訊息ID
  int64 Timestamp; // 時間戳記
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 玩家尚未登入
    -3: 無法取得聊天室
    -4: 只能加入私人聊天室或公頻
    -5: 聊天室人數已滿
    -6: 已在聊天室
    -7: 無法加入聊天室
```

## Api - chat.LeaveChat 離開聊天室

```bash
LeaveChatRequest:
  string ChatId; // 聊天室ID
```

```bash
LeaveChatResponse:
```

```bash
LeaveChatReceipt:
  string ChatId; // 聊天室ID
  string MemberId; // 聊天室成員ID(玩家ID)
  string MessageId; // 訊息ID
  int64 Timestamp; // 時間戳記
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 玩家尚未登入
    -3: 無法取得聊天室
    -4: 只能離開私人聊天室或公頻
    -5: 不在聊天室
    -6: 無法離開聊天室
```

## Api - chat.SendText 傳送文字訊息

```bash
SendTextRequest:
  string To; // 傳送給誰. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室的ID
  string Text; // 文字內容
  string RelatedMessageId; // 回覆訊息ID. 如果是要回覆某筆訊息, 此欄位為該筆訊息的ID, 否則為空字串
```

```bash
SendTextResponse:
  string MessageId; // 訊息ID. 可以用來比對SendTextReceipt.MessageId
```

```bash
SendTextReceipt:
  string From; // 訊息發送者ID
  string To; // 訊息接收者ID. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室的ID
  string MessageId; // 訊息ID
  int64 Timestamp; // 時間戳記
  string RelatedMessageId; //回覆訊息ID
  string Text; // 文字內容
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 玩家尚未登入
    -3: 無法取得聊天室
    -4: 訊息傳送者不可為訊息接收者
    -5: 訊息傳送者不在聊天室
    -6: 訊息接收者不在聊天室
    -16384: 內部錯誤
```

## Api - chat.SendImage 傳送影像訊息

```bash
SendImageRequest:
  string To; // 傳送給誰. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室的ID
  string RelatedMessageId; // 回覆訊息ID. 如果是要回覆某筆訊息, 此欄位為該筆訊息的ID, 否則為空字串
```

```bash
SendImageResponse:
  string MessageId; // 訊息ID. 可以用來比對SendTextReceipt.MessageId
  string ContentUrl; // 上傳影像的Url
```

```bash
SendImageReceipt:
  string From; // 訊息發送者ID
  string To; // 訊息接收者ID. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室的ID
  string MessageId; // 訊息ID
  int64 Timestamp; // 時間戳記
  string RelatedMessageId; //回覆訊息ID
  string ContentUrl; // 上傳影像的Url
```

```bash
SendImageCheckedReceipt:
  string From; // 訊息發送者ID
  string To; // 訊息接收者ID. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室的ID
  string MessageId; // 訊息ID. 這是影像已上傳完畢所產生的訊息ID
  int64 Timestamp; // 時間戳記
  string OriginalMessageId; // 訊息ID. 這是原本SendImageRequest所產生的訊息ID, 也就是SendImageResponse.MessageId
  string ContentUrl; // 下載影像的Url
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 玩家尚未登入
    -3: 無法取得聊天室
    -4: 訊息傳送者不可為訊息接收者
    -5: 訊息傳送者不在聊天室
    -6: 訊息接收者不在聊天室
    -16384: 內部錯誤
```

## Api - chat.FetchMessage 獲取聊天室訊息

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
SendImageRequest:
  string To; // 傳送給誰. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室的ID
```

```bash
SendImageResponse:
```

```bash
SendImageReceipt:
  string From; // 訊息發送者ID
  string To; // 訊息接收者ID. 如果是私聊, 此欄位為玩家ID, 其餘為聊天室的ID
  string MessageId; // 訊息ID. 這是影像已上傳完畢所產生的訊息ID
  int64 Timestamp; // 時間戳記
  string LastReadMessageId; // 最後一筆已讀訊息ID. 這表示在這之前的訊息ID都是已讀的
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 玩家尚未登入
    -3: 無法取得聊天室
    -4: 訊息傳送者不可為訊息接收者
    -5: 訊息傳送者不在聊天室
    -6: 訊息接收者不在聊天室
    -7: 已讀狀態只支援私聊
    -16384: 內部錯誤
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
  bytes ReceiptMessage; // 對應ReceiptMessage結構
```
