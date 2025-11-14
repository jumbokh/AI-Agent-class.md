

---

## **實驗 6 流程圖指導說明**

### **流程圖解讀與教學重點**

**核心架構：多執行緒 + 事件驅動 + Web 服務**

**步驟 C：啟動 Flask Web Server**
- **技術架構**：
  ```python
  # 主執行緒：Flask Web 服務
  # 子執行緒：卡片輪詢監聽
  Thread(target=poller, daemon=True).start()  # 後台輪詢
  app.run(host="0.0.0.0", port=5000)          # 主Web服務
  ```
- **教學重點**：
  - **Daemon Thread**：確保程式結束時自動清理
  - **執行緒安全**：共享資料 `events` 列表的存取保護

### **卡片監聽與事件處理流程**

**「連接讀卡機並監聽卡片」**
- **輪詢機制**：
  ```python
  def poller():
      seen = None  # 記錄上次檢測到的 UID
      while True:
          uid = get_current_uid()
          if uid and uid != seen:  # 新卡片出現
              process_new_card(uid)
              seen = uid
          elif not uid:  # 卡片移出
              seen = None
          sleep(0.5)  # 避免 CPU 過載
  ```

**「取得 UID」與狀態管理**
- **防抖機制**：避免同一卡片重複觸發
- **狀態追蹤**：`seen` 變數記錄當前在場卡片

### **數據解析與格式化**

**「資料為 BAL=格式？」決策邏輯**
```python
def parse_balance_data(raw_data):
    """解析 Block 4 的 BAL= 格式數據"""
    try:
        # 轉換為字串並清理
        text_data = bytes(raw_data).decode('latin1', 'ignore').strip('\x00 ')
        
        if text_data.startswith("BAL="):
            balance = int(text_data.split("=")[1])
            return balance
        else:
            return None  # 非 BAL 格式
    except (ValueError, IndexError):
        return None  # 解析失敗
```

**事件記錄結構：**
```python
event_record = {
    'timestamp': '2024-03-20 14:30:25',
    'uid': '04 5A 3B 7C',
    'balance': '150',  # 可能為空字串
    'type': 'wallet' if balance else 'access'  # 事件類型分類
}
```

### **Flask Web 服務設計**

**網頁模板設計：**
```html
<!doctype html>
<meta charset="utf-8">
<title>NFC 刷卡監控系統</title>
<style>
    table { border-collapse: collapse; width: 100%; }
    th, td { border: 1px solid #ccc; padding: 8px; text-align: left; }
    tr:nth-child(even) { background-color: #f2f2f2; }
</style>
<h1>📊 NFC 刷卡事件監控</h1>
<p>最近 20 筆刷卡記錄（自動更新）</p>
<table>
    <tr><th>時間</th><th>卡片 UID</th><th>餘額</th><th>類型</th></tr>
    {% for event in events %}
    <tr>
        <td>{{ event.ts }}</td>
        <td><code>{{ event.uid }}</code></td>
        <td>{{ event.bal if event.bal else "N/A" }}</td>
        <td>{{ "錢包" if event.bal else "門禁" }}</td>
    </tr>
    {% endfor %}
</table>
<script>
// 每 3 秒自動刷新頁面
setTimeout(() => location.reload(), 3000);
</script>
```

### **生物聯網系統整合應用**

**實驗室設備管理場景：**
```
事件記錄示例：
1. 2024-03-20 10:15:30 | 04 5A 3B 7C | 85點 | 顯微鏡使用扣款
2. 2024-03-20 10:16:15 | 04 8C 2D 9A | N/A   | 實驗室門禁進入
3. 2024-03-20 10:17:22 | 04 5A 3B 7C | 80點 | 離心機使用扣款
```

**多讀卡器擴展設計：**
```python
# 支持多個讀卡器的架構
readers_config = {
    'reader1': {'location': '實驗室大門', 'type': 'access'},
    'reader2': {'location': '顯微鏡工作站', 'type': 'billing'}, 
    'reader3': {'location': '藥品櫃', 'type': 'inventory'}
}
```

### **錯誤處理與穩定性設計**

**讀卡器異常恢復：**
```python
def resilient_poller():
    while True:
        try:
            # 正常的輪詢邏輯
            poll_once()
        except Exception as e:
            print(f"輪詢異常: {e}")
            reconnect_reader()  # 嘗試重新連接
            sleep(5)  # 等待後重試
```

**記憶體管理與性能優化：**
```python
# 限制事件數量，避免記憶體洩漏
if len(events) >= 20:
    events.pop()  # 移除最舊的事件
events.insert(0, new_event)  # 新增事件到開頭
```

### **學生實作任務**

**基礎功能實現：**
1. ✅ 完成基本的輪詢與 Web 顯示
2. ✅ 正確解析 BAL= 格式與普通 UID
3. ✅ 實現自動刷新與事件記錄

**進階功能挑戰：**
1. 🎯 **數據持久化**：將事件保存到 SQLite 資料庫
2. 🎯 **RESTful API**：提供 JSON API 供其他系統調用
3. 🎯 **即時推送**：使用 WebSocket 替代自動刷新
4. 🎯 **多讀卡器**：支持同時監控多個 ACR122U

**擴展應用開發：**
```python
# API 端點示例
@app.route('/api/events')
def api_events():
    return jsonify(events)

@app.route('/api/cards')
def api_cards():
    # 返回所有已知卡片的統計信息
    return jsonify(analyze_cards(events))
```

### **系統部署與測試**

**跨平台測試指南：**
- **Windows**：使用 `py app.py` 啟動服務
- **Raspberry Pi**：使用 `python3 app.py` 啟動
- **網路訪問**：在同區域網路其他設備用 `http://[IP]:5000` 訪問

**性能優化建議：**
- 調整輪詢間隔：0.5-1.0 秒平衡響應與 CPU 使用
- 使用生產級 WSGI 伺服器（如 gunicorn）
- 添加日誌記錄以便故障排查

---

這個流程圖完美展示了**從邊緣設備到 Web 應用的完整物聯網架構**。
