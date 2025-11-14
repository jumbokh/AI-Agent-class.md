

---

## **實驗 4 流程圖指導說明**

### **流程圖解讀與教學重點**

**核心流程：掃描頁面 → 尋找 TLV → 解析 NDEF → 顯示內容**

**步驟 B：從 Page 4 開始逐頁讀取**
- **技術解析**：
  - NTAG 記憶體結構：Page 0-1 = UID, Page 2 = Capability Container, Page 3 = 初始狀態
  - **Page 4 開始**才是用戶可用的數據區域
- **教學重點**：
  - 為什麼從 Page 4？避免讀取到系統保留區域
  - 頁讀取 APDU：`FF B0 00 <page> 04`（每次讀取 4 位元組）
- **生物聯網意義**：標準化的記憶體布局有利於跨設備兼容

**關鍵步驟：檢查 TLV 結構 (Tag-Length-Value)**
- **TLV 格式詳解**：
  ```
  [0x03] [Length] [NDEF Message...] [0xFE]
  ↑      ↑        ↑                 ↑
  Tag    Length   Value             Terminator
  ```
- **教學重點**：
  - `0x03` = NDEF Message TLV
  - `0xFE` = Terminator TLV（結束標記）
  - `0x00` = Null TLV（填充）

**決策點：有找到 NDEF TLV？**
- **可能情況**：
  - **找到 `0x03`**：繼續解析 NDEF 內容
  - **找到 `0xFE`**：已到達數據結尾，停止掃描
  - **找到 `0x00`**：空白區域，繼續掃描
  - **其他值**：非 NDEF 數據，根據長度跳過

### **NDEF 解析深度教學**

**步驟 G：判斷是否為文字記錄**
- **NDEF 記錄頭解析**：
  ```python
  # 示例: D1 01 0A 54 02 65 6E 48 65 6C 6C 6F
  # D1 = 11010001 (MB=1, ME=1, CF=0, SR=1, IL=0, TNF=001)
  # 01 = 類型長度 = 1
  # 0A = 負載長度 = 10
  # 54 = 類型 = 'T' (Text)
  # 02 = 狀態位元組 (語言碼長度=2)
  # 65 6E = 語言碼 "en"
  # 48 65 6C 6C 6F = 文本 "Hello"
  ```
- **教學重點**：
  - TNF (Type Name Format) = 001 = Well-Known Type
  - 類型 `54` = 'T' = Text Record
  - SR (Short Record) = 1 → 負載長度用 1 位元組表示

**文字記錄解析算法：**
```python
def parse_text_record(ndef_payload):
    if len(ndef_payload) < 3:
        return None
    
    # 檢查記錄頭
    if (ndef_payload[0] & 0xD8) != 0xD1:  # MB+ME+SR+TNF
        return None
    if ndef_payload[3] != 0x54:  # 類型不是 'T'
        return None
    
    payload_length = ndef_payload[2]
    payload = ndef_payload[4:4 + payload_length]
    
    # 解析狀態位元組和語言碼
    status_byte = payload[0]
    lang_length = status_byte & 0x3F
    text_data = payload[1 + lang_length:]
    
    return text_data.decode('utf-8', 'ignore')
```

### **錯誤處理與特殊情況**

**「未找到 NDEF 訊息」**
- **可能原因**：
  - 卡片是空的（全 0x00）
  - 包含非 NDEF 格式的數據
  - TLV 結構損壞或不標準
- **解決方案**：使用 NFC Tools 等應用檢查卡片實際內容

**「非文字記錄」**
- **其他常見記錄類型**：
  - **URI 記錄**：類型 = 'U' (0x55)，如網址、電話號碼
  - **智能海報**：包含多條記錄的複雜結構
  - **MIME 類型**：自定義二進位數據

### **生物聯網應用場景**

**樣本管智能標籤：**
```python
# NDEF 內容示例：標本ID + 採集時間
ndef_text = "SAMPLE:BLD-2024-001\nTIME:2024-03-20 10:30\nSTATUS:PROCESSED"
```

**設備維護記錄標籤：**
```python
# 包含設備信息和維護歷史
ndef_text = "DEVICE:Microscope-X100\nLAST_SERVICE:2024-02-15\nNEXT_SERVICE:2024-05-15"
```

### **學生實作任務**

**基礎任務：**
1. 成功掃描並找到 NDEF TLV
2. 解析並顯示 Text Record 內容
3. 處理空卡片和非文字記錄情況

**進階挑戰：**
1. 擴展支持 URI Record 解析
2. 實現 NDEF 寫入功能（創建新的 TLV 結構）
3. 設計二進位數據的 MIME 類型記錄

**除錯技巧：**
- 逐頁顯示十六進位內容，觀察 TLV 結構
- 使用 `toHexString(list(ndef))` 顯示完整的 NDEF 數據
- 對比不同 NFC 應用寫入的卡片內容

### **實用工具推薦**

**驗證工具：**
- **NFC Tools (手機 App)**：快速驗證卡片內容
- **線上 NDEF 解析器**：分析複雜的 NDEF 結構
- **十六進位編輯器**：深入查看原始數據

---

這個流程圖清晰地展示了**數據發現與解析**的完整過程。
