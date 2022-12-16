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
        101: [0]: 目標步數
        102: [0]: 目標里程
      揪一起:
        201: [0] 定位GPS位置

    string[] dataList // 任務資料. 透過updateTask更新
      揪運動: 
        101: [0]: 所有玩家總步數
        102: [0]: 所有玩家總里程
      揪一起: 
        201: [0]: 第幾階段(階段1: 到達定位位置, 階段2: 聊天100句)

    list<{
      string userId, // 玩家ID
      string[] dataList // 玩家資料
        揪運動:
          101: [0]: GPS位置, [1]: 玩家個人總步數
          102: [0]: GPS位置, [1]: 玩家個人總里程
        揪一起:
          201: [0]: 確認到達
    }> userList; // 透過updateTask更新

    int status: 任務狀態. 0: 尚未開始, 1:進行中, 2:已結束(任務聊天室還在)
    int64 dueDate; 任務失效日期(timestamp格式)
    int64 completedDate; 任務結束時間(timestamp格式). 0: 尚未結束
    int autoDismissSeconds; 任務結束後的解散時間(秒數). 0: 不解散
```

## 功能: updateTask 更新任務

```bash
request:
  string taskId; 聊天室(任務)ID
  string[] taskDataList // 更新任務資料
  list<{
    string userId, // 玩家ID
    string[] dataList // 更新玩家資料
  }> userList;
```

```bash
response:
  int error; 0: 成功, -1: taskId不存在, -2: task已結束(任務聊天室還在)

  case error == 0:
    list<string> invalidUserIdList; 無效的筆數(userId不在聊天室). 這裡假設資料都是正確的, 所以只回傳有誤的
```

## 功能: completeTask 完成任務

> userList 只包含在線玩家

```bash
request:
  string taskId; 聊天室(任務)ID
  list<{
    string userId // 玩家ID
  }> userList;
```

```bash
response:
  int error; 0: 成功, -1: taskId不存在, -2: task已結束

  case error == 0:
    list<string> invalidUserIdList; 無效的筆數(userId不在聊天室). 這裡假設資料都是正確的, 所以只回傳有誤的
```
