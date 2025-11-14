

---

## **實驗 5 流程圖指導說明**

### **流程圖解讀與教學重點**

**核心流程：讀取→備份→修改→寫入→驗證**

**步驟 B：讀取 Sector Trailer（最後一塊）**
- **技術關鍵**：
  - Sector Trailer = 塊 3, 7, 11, 15...（每個扇區的最後一塊）
  - 結構：`[Key A (6B)] [Access Bits (4B)] [Key B (6B)]`
- **教學重點**：
  - **必須使用已知金鑰**讀取 Trailer
  - 讀取後立即備份，作為恢復依據
- **安全警示**：這是整個實驗中最危險的操作之一

### **Sector Trailer 深度解析**

**存取控制位元組計算（C1 C2 C3）：**
```python
# 存取控制位元組結構（每個區塊 3 個控制位）
# 位元意義：0=Key A, 1=Key B, 2=Key A|B, 3=Never
access_bits_example = {
    'data_blocks': [2, 2, 2, 2],  # Key A|B 用於讀寫
    'sector_trailer': [0, 0, 1]   # Key A 用於讀存取控制，Key B 不可用
}

# 計算 C1, C2, C3 位元組
def calculate_access_bits(block_bits):
    # 複雜的位元映射計算（實際使用應參考官方文檔）
    c1, c2, c3 = 0xFF, 0x07, 0x80  # 示例值
    return [c1, c2, c3, 0x00]  # 4 位元組存取控制
```

**新的 Trailer 資料構建：**
```python
def build_new_trailer(new_key_a, access_bits, new_key_b):
    """
    構建新的 Sector Trailer
    new_key_a: [0xA0, 0xA1, 0xA2, 0xA3, 0xA4, 0xA5]
    access_bits: [0xFF, 0x07, 0x80, 0x00] 
    new_key_b: [0xB0, 0xB1, 0xB2, 0xB3, 0xB4, 0xB5]
    """
    trailer_data = new_key_a + access_bits + new_key_b
    assert len(trailer_data) == 16, "Trailer 必須是 16 位元組"
    return trailer_data
```

### **關鍵風險點與安全措施**

**「一次性寫入」的極端重要性：**
- **風險**：重複寫入可能損壞存取控制位元
- **保護措施**：
  ```python
  # 在寫入前進行多重驗證
  assert len(new_trailer) == 16
  assert validate_access_bits(access_bits)  # 驗證存取控制位元有效性
  ```

**「卡片可能鎖死」的嚴重後果：**
- **症狀**：該扇區永久無法讀寫
- **預防**：
  1. 僅在**可丟棄的空白卡**上操作
  2. 操作前完整備份所有扇區
  3. 使用經過驗證的存取控制位元組合

### **錯誤處理與恢復策略**

**「寫入失敗」的可能原因：**
- 認證已過期
- 存取控制位元計算錯誤
- 硬體寫入保護

**「新金鑰認證失敗」的處理：**
1. **嘗試原金鑰**：確認扇區是否還可訪問
2. **檢查 Trailer 內容**：讀取驗證實際寫入的數據
3. **系統性排查**：金鑰順序、存取控制位元、認證參數

### **教學用安全存取控制設定**

**推薦的教學設定：**
```python
# 數據塊：Key A 可讀寫，Key B 無權限
# Sector Trailer：Key A 可讀存取控制，Key B 不可用
safe_access_bits = [0xFF, 0x07, 0x80, 0x00]

# 對應的權限：
# - 數據塊: Key A 可讀/寫，Key B 無權限  
# - Trailer: Key A 可讀存取控制，Key B 無權限
```

### **生物聯網安全應用**

**多層級權限管理系統：**
```python
# 不同設備的不同權限等級
permission_levels = {
    'admin': {'key_a': [0xA0,0xA1,0xA2,0xA3,0xA4,0xA5], 'access': [0xFF,0x07,0x80,0x00]},
    'operator': {'key_a': [0xB0,0xB1,0xB2,0xB3,0xB4,0xB5], 'access': [0xFF,0x07,0x88,0x00]},
    'reader': {'key_a': [0xC0,0xC1,0xC2,0xC3,0xC4,0xC5], 'access': [0xFF,0x07,0x80,0x00]}
}
```

### **學生實作指導**

**必須遵守的安全守則：**
1. ✅ **僅使用空白測試卡**
2. ✅ **操作前完整備份**
3. ✅ **一次寫入，絕不重試**
4. ✅ **立即驗證，及時止損**

**實作步驟：**
1. **準備階段**：選擇扇區 1（塊 4-7）進行實驗
2. **備份階段**：讀取並記錄原始 Trailer 內容
3. **計算階段**：使用安全的存取控制位元組合
4. **執行階段**：一次性寫入新 Trailer
5. **驗證階段**：用新金鑰認證並讀取數據

**除錯腳本：**
```python
def debug_sector_trailer(conn, sector, key_a):
    """診斷扇區狀態"""
    trailer_block = sector * 4 + 3
    try:
        auth(conn, trailer_block, key_a)
        data, sw1, sw2 = read_block(conn, trailer_block)
        print(f"Sector {sector} Trailer: {toHexString(data)}")
        return True
    except Exception as e:
        print(f"Sector {sector} 可能已鎖死: {e}")
        return False
```

### **緊急恢復方案**

**如果操作失敗：**
1. **立即停止**：不要嘗試任何修復操作
2. **更換卡片**：使用新的空白卡重新開始
3. **分析原因**：檢查備份數據和操作日志
4. **調整策略**：簡化存取控制位元設定

---

這個流程圖強調了**安全操作的最高原則**。
