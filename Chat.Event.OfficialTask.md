## Api -> Server (-> App)

## 功能: officialTaskCreated 官方任務已創建

```bash
request:
  string taskId; 聊天室(任務)ID
```

```bash
response:
  int error: 0: 成功, -1: 失敗
  string message: 錯誤訊息
```
