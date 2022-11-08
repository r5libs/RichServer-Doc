## 如何呼叫 Server Api?

```bash
[方式1]
xxxResponse response = await SocketChannel.SendRequest<xxxRequest, xxxResponse>("xxx.yyy", new xxxRequest()
{

});

[方式2]
SocketChannel.SendRequest("xxx.yyy", new xxxRequest()
{

}, (Exception error, xxxResponse response) =>
{
  if (error != null)
  {

  }
  else
  {

  }
});
```

## 如何接收由 Server 主動通知的訊息?

```bash
在SocketChannel.Message加入訊息處理函式, 以下為可能的messageType列表:
  chat.SendMessageFailed // 當SendText/SendImage等Api執行失敗時
  chat.SendReceiptMessage // 當聊天室內容有更動時
```

## 如何取得 MessageResponseException 的 ErrorCode

```bash
每個Server Api若執行異常均會丟出MessageResponseException例外, 以下為提供的二種方式:

[方式1]
try
{
  xxxResponse response = await SocketChannel.SendRequest<xxxRequest, xxxResponse>("xxx.yyy", new xxxRequest()
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
SocketChannel.SendRequest("xxx.yyy", new xxxRequest()
{

}, (Exception error, xxxResponse response) =>
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

## login.VerifyUser 登入驗證

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

## chat.UseChat 使用聊天服務

```bash
UseChatRequest:
  string ChatKey; // 可經由login.VerifyUser取得
```

```bash
UseChatResponse:
  uint64 LastOpRevision; // 最後一筆訊息紀錄
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

## chat.JoinChat 加入聊天室

```bash
JoinChatRequest:
  string ChatId;
```

```bash
JoinChatResponse:
```

```bash
JoinChatReceipt:
  string ChatId;
  string MemberId;
  string MessageId;
  int64 Timestamp;
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

## chat.LeaveChat 離開聊天室

```bash
LeaveChatRequest:
  string ChatId;
```

```bash
LeaveChatResponse:
```

```bash
LeaveChatReceipt:
  string ChatId;
  string MemberId;
  string MessageId;
  int64 Timestamp;
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

## chat.SendText 傳送文字訊息

```bash
SendTextRequest:
  string To;
  string Text;
  string RelatedMessageId;
```

```bash
SendTextResponse:
  string MessageId;
```

```bash
SendTextReceipt:
  string From;
  string To;
  string MessageId;
  int64 Timestamp;
  string RelatedMessageId;
  string Text;
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

## chat.SendImage 傳送影像訊息

```bash
SendImageRequest:
  string To;
  string RelatedMessageId;
```

```bash
SendImageResponse:
  string MessageId;
  string ContentUrl;
```

```bash
SendImageReceipt:
  string From;
  string To;
  string MessageId;
  int64 Timestamp;
  string RelatedMessageId;
  string ContentUrl;
```

```bash
SendImageCheckedReceipt:
  string From;
  string To;
  string MessageId;
  int64 Timestamp;
  string RelatedMessageId;
  string ContentUrl;
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

## chat.FetchMessage 獲取聊天室訊息

```bash
FetchMessageRequest:
  uint64 Start;
  int32 Count;
```

```bash
FetchMessageResponse:
  UserReceiptMessage[] Messages;
```

```bash
MessageResponseException:
  int ErrorCode;
    -1: 請求參數無效
    -2: 玩家尚未登入
    -3: Start大於實際最大值
    -4: Count太大
```

## chat.SendMessageRead 告知聊天室訊息已讀

```bash
SendImageRequest:
  string To;
```

```bash
SendImageResponse:
```

```bash
SendImageReceipt:
  string From;
  string To;
  string MessageId;
  int64 Timestamp;
  string LastReadMessageId;
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
