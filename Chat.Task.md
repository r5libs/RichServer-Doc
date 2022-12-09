## Server -> Api

## 功能: getTask 取得任務

```bash
request:
  string taskId; 聊天室(任務)ID
```

```bash
response:
  int error; 0: 成功, -1: chatId不存在

  case error == 0:
    int type; 任務類型. 1: 揪運動, 2: 揪一起
    int mode; 任務模式
      揪運動: 101: 步數, 102: 里程
      揪一起: 201: 定位
    int[] meetConditions; 任務達成條件列表
      揪運動: [0]: 步數/里程目標
```

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

  case taskType == 1: // 揪運動
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
