## Server -> Api

## 功能: getTask 取得任務

```bash
request:
  string taskId; 聊天室(任務)ID
```

```bash
response:
  int error; 0: 成功, -1: taskId不存在

  case error == 0:
    int type; 任務類型. 1: 揪運動, 2: 揪一起
    int mode; 任務模式
      揪運動: 101: 步數, 102: 里程
      揪一起: 201: 定位+聊天100句

    string[] meetConditions; 任務達成條件列表
      揪運動:
        101: [0]: 目標步數, [1]: 失效日期(群主沒有下達任務開始該任務就算失效), [2]: 結束時間(任務沒完成就算失敗)
        102: [0]: 目標里程, [1]: 失效日期(群主沒有下達任務開始該任務就算失效), [2]: 結束時間(任務沒完成就算失敗)
      揪一起:
        201: [0]: 目標GPS位置, [1]: 報到時間(玩家到達指定地點才可以報到. 所有人都報到可忽略開始時間/報到人數須>=2/有報到的一定要確認開始), [2]: 開始時間(報到後, 玩家才可以開始任務(下一階段)) [3]: 結束時間(任務沒完成就算失敗)

    string[] dataList; 任務資料. 透過updateTask更新
      揪運動:
        101: [0]: 所有玩家總步數
        102: [0]: 所有玩家總里程
      揪一起:
        201: [0]: 0: 等候報到, 1: 等候開始(下一階段), 2: 進行聊天100句, [1]: 所有玩家總聊天句數, [2]: 正式開始的時間(此時[0]的值是"2")

    list<{
      string userId, 玩家ID
      string[] dataList 玩家資料
        揪運動:
          101: [0]: GPS位置, [1]: 玩家個人總步數
          102: [0]: GPS位置, [1]: 玩家個人總里程
        揪一起:
          201: [0]: 已確認報到(0 or 1), [1]: 已確認開始(0 or 1), [2]: 玩家個人聊天總句數, [3]: GPS位置
    }> userList; 透過updateTask更新

    int status: 任務狀態. 0: 尚未開始, 1: 進行中, 2: 已結束(失敗), 3: 已結束(成功), 4: 已取消, 5: 已結束(autoDismissSeconds)
    int autoDismissSeconds; 任務結束後的解散時間(秒數). 0: 不解散
```

## 功能: updateTask 更新任務

```bash
request:
  string taskId; 聊天室(任務)ID
  string[] taskDataList; 任務資料
  list<{
    string userId, 玩家ID
    string[] dataList 玩家資料
  }> userList;
```

```bash
response:
  int error; 0: 成功, -1: taskId不存在, -2: task已結束

  case error == 0:
    list<string> invalidUserIdList; 無效的筆數(userId不在聊天室). 這裡假設資料都是正確的, 所以只回傳有誤的
```

## 功能: completeTask 完成任務

> userList 只包含在線玩家

```bash
request:
  string taskId; 聊天室(任務)ID
  int status; 2: 已結束(失敗), 3: 已結束(成功), 4: 已取消, 5: 已結束(autoDismissSeconds)
  list<{
    string userId // 玩家ID
  }> userList;
```

```bash
response:
  int error; 0: 成功, -1: taskId不存在, -2: task已結束

  case error == 0:
    list<string> invalidUserIdList; 無效的筆數(userId不在聊天室). 這裡假設資料都是正確的, 所以只回傳有誤的

    case status == 3: 
      int rewardType; 任務完成時的獎勵(只有當status為3時, 此值才有意義)
      int tokenType; 代幣類型. 0: Rock幣, 1: 水晶碎片(只有當status為3時, 此值才有意義)
```
