## 功能: completeTask

```bash
- request:
  string taskId; 聊天室(任務)ID
  int taskType; 1: 揪運動
  list<{string userId}> userList; 當時在線的玩家列表
```

```bash
- response:
  int error; 0: 成功
```
