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
  - chat.SendMessageFailed // 當SendText/SendImage等Api執行失敗時
  - chat.SendReceiptMessage // 當聊天室內容有更動時
```

## 如何取得 MessageResponseException 的 ErrorCode

```bash
若Server Api執行異常, 則丟出MessageResponseException例外, 以下為提供的二種方式:

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

## chat.LeaveChat 離開聊天室

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

## chat.SendText 傳送文字訊息

```bash
SendTextRequest:
  string To; // 傳送給誰. 如果是私聊, 此欄位為玩家ID, 其餘為該聊天室的ID
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
  string To; // 訊息接收者ID. 如果是私聊, 此欄位為玩家ID, 其餘為該聊天室的ID
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

## chat.SendImage 傳送影像訊息

```bash
SendImageRequest:
  string To; // 傳送給誰. 如果是私聊, 此欄位為玩家ID, 其餘為該聊天室的ID
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
  string To; // 訊息接收者ID. 如果是私聊, 此欄位為玩家ID, 其餘為該聊天室的ID
  string MessageId; // 訊息ID
  int64 Timestamp; // 時間戳記
  string RelatedMessageId; //回覆訊息ID
  string ContentUrl; // 上傳影像的Url
```

```bash
SendImageCheckedReceipt:
  string From; // 訊息發送者ID
  string To; // 訊息接收者ID. 如果是私聊, 此欄位為玩家ID, 其餘為該聊天室的ID
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

## chat.FetchMessage 獲取聊天室訊息

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

## chat.SendMessageRead 傳送訊息已讀通知

```bash
SendImageRequest:
  string To; // 傳送給誰. 如果是私聊, 此欄位為玩家ID, 其餘為該聊天室的ID
```

```bash
SendImageResponse:
```

```bash
SendImageReceipt:
  string From; // 訊息發送者ID
  string To; // 訊息接收者ID. 如果是私聊, 此欄位為玩家ID, 其餘為該聊天室的ID
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
