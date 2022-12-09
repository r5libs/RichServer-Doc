## Server -> Api

## 功能: completeTask 完成任務

```bash
- request:
  string taskId; 聊天室(任務)ID
  list<{
    string userId // 玩家ID
  }> dataList;
```

```bash
- response:
  int error; 0: 成功
```

## 功能: updateTask 更新任務

```bash
- request:
  string taskId; 聊天室(任務)ID

  case taskType == 揪運動:
    list<{
      string userId, // 玩家ID
      string location, // GPS位置
      int move // 移動量(步數/里程)
    }> dataList;
```

```bash
- response:
  int error; 0: 成功
```
