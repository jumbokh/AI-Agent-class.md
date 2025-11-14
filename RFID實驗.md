好的，這是完整的**教師版講義**內容。您可以直接將以下內容複製到 Word 文件中進行排版和列印。

---

# **ACR122U NFC 讀卡器與 Python 物聯網應用實作 - 教師版講義**

**適用對象：** 本科生物聯網系
**課程時數：** 2-4 次實驗課 (每次 3 小時)
**授課教師：** _________________________
**課程日期：** _________________________

---

## **目錄**
1. 課程總覽與準備工作
2. 實驗 1：連線、偵測、讀取 ATR 與 UID
3. 實驗 2：MIFARE Classic 讀區塊
4. 實驗 3：MIFARE Classic 寫入與「小額錢包」
5. 實驗 4：NTAG/Ultralight 讀寫與 NDEF
6. 實驗 5：金鑰管理（高階操作）
7. 實驗 6：Flask 即時點名/資產盤點系統
8. 評分標準與解答
9. 附錄：APDU 指令速查表

---

## **1. 課程總覽與準備工作**

### **實驗總目標**
* 熟悉 ACR122U 讀卡流程（PC/SC、ATR、UID）
* 能讀寫 **MIFARE Classic 1K/4K** 指定區塊（僅限自有空白卡、預設金鑰）
* 能讀寫 **NTAG/Ultralight（Type 2）** 的 NDEF（Text/URL）
* 完成一個「簡易門禁/點名/小額錢包」的 Python 小系統

### **硬體與耗材**
* ACR122U 讀卡機（USB）
* 樣本卡：
  - MIFARE Classic 1K（S50）× 2（空白、預設 Key A = FF FF FF FF FF FF）
  - NTAG213/215/216 × 1（內建 NDEF 區）
* 電腦：Windows 10/11 或 Raspberry Pi（Raspberry Pi OS）

### **系統安裝**

**Windows 系統：**
```powershell
# 安裝 Python 套件
py -m pip install pyscard

# 驗證安裝
py -c "from smartcard.System import readers; print(readers())"
```

**Raspberry Pi 系統：**
```bash
sudo apt-get update
sudo apt-get install -y pcscd pcsc-tools libpcsclite-dev python3-pyscard
sudo systemctl enable --now pcscd
pcsc_scan  # 掃描讀卡機與卡片
```

### **安全規範與注意事項**
1. **僅操作自有或授權的卡片與金鑰**
2. **不得嘗試繞過門禁/支付系統**
3. **先從空白卡、預設金鑰開始**
4. **勿動製造商區（Block 0）和扇區尾（Sector Trailer）**

---

## **2. 實驗 1：連線、偵測、讀取 ATR 與 UID**

### **實驗目標**
- 會列出讀卡機、連線卡片、取得 ATR 與 UID
- 理解 PC/SC 架構基礎

### **程式碼實作**
```python
# exp1_uid.py
from smartcard.System import readers
from smartcard.util import toHexString

GET_UID = [0xFF, 0xCA, 0x00, 0x00, 0x00]  # ACR122U: Get Data (UID)

r = readers()
if not r:
    raise SystemExit("找不到讀卡機，請檢查驅動/pcscd。")
print("Readers:", r)

conn = r[0].createConnection()
conn.connect()  # 自動選卡
atr = conn.getATR()
print("ATR:", toHexString(atr))

data, sw1, sw2 = conn.transmit(GET_UID)
if (sw1, sw2) == (0x90, 0x00):
    print("UID:", toHexString(data))
else:
    print(f"讀 UID 失敗: {sw1:02X} {sw2:02X}")
```

### **教師解答與核心知識點**

**Q: ATR (Answer To Reset) 是什麼？**
- **解答：** 當卡片被上電或重置時，發送給讀卡器的一串字節。它包含了卡片的基本信息，如支持的通信協議、製造商信息等。這是一個底層的、標準化的身份標識。

**Q: UID (Unique Identifier) 長度為何會有 4/7 位元組差異？**
- **解答：** UID 的長度由卡片標準和製造商決定。
  - **4 位元組：** 最常見的 MIFARE Classic 卡片使用 4 位元組 UID
  - **7 位元組：** 遵循 ISO/IEC 14443-3 標準的卡片（如 NTAG、DESFire）使用 7 位元組 UID
- **生物聯網關聯：** 在設計系統時，必須預先考慮 UID 長度的可變性，數據庫設計應能容納不同長度的 UID

### **測試點**
- 能穩定印出 ATR 與 UID（十六進位）
- 能解釋 ATR 和 UID 的差異

---

## **3. 實驗 2：MIFARE Classic 讀區塊**

### **實驗目標**
- 能「載入金鑰 → 驗證扇區 → 讀取資料區塊」
- 理解 Classic 記憶體結構（16 bytes/block，4 blocks/sector）

### **程式碼實作**
```python
# exp2_mfc_read.py
from smartcard.System import readers
from smartcard.util import toHexString

# PC/SC 擴充：ACR122U 對 MIFARE 的常用 APDU
LOAD_KEY = lambda slot, key: [0xFF, 0x82, 0x00, slot, 0x06] + key
AUTHENTICATE = lambda block, key_type, slot: [0xFF, 0x86, 0x00, 0x00, 0x05, 0x01, 0x00, block, key_type, slot]
READ_BLOCK = lambda block: [0xFF, 0xB0, 0x00, block, 0x10]  # 16 bytes

KEY_A = [0xFF] * 6
KEY_SLOT = 0x00
KEY_TYPE_A = 0x60  # 0x60=Key A, 0x61=Key B
BLOCK = 4  # 扇區1的第一塊(通常可用資料區)

conn = readers()[0].createConnection()
conn.connect()

# 載入 Key A 到插槽
data, sw1, sw2 = conn.transmit(LOAD_KEY(KEY_SLOT, KEY_A))
assert (sw1, sw2) == (0x90, 0x00), f"載入金鑰失敗: {sw1:02X} {sw2:02X}"

# 驗證該區塊所屬扇區（用 Key A）
data, sw1, sw2 = conn.transmit(AUTHENTICATE(BLOCK, KEY_TYPE_A, KEY_SLOT))
assert (sw1, sw2) == (0x90, 0x00), f"認證失敗: {sw1:02X} {sw2:02X}"

# 讀取 16 bytes
data, sw1, sw2 = conn.transmit(READ_BLOCK(BLOCK))
if (sw1, sw2) == (0x90, 0x00):
    print(f"Block {BLOCK}:", toHexString(data), " | ASCII:", bytes(data).decode('latin1', 'ignore'))
else:
    print(f"讀取失敗: {sw1:02X} {sw2:02X}")
```

### **教師解答與核心知識點**

**Q: 為什麼要先「載入金鑰」再「認證」？**
- **解答：** 這是 ACR122U 讀卡器設計的安全與效率折衷方案
  1. **載入金鑰 (FF 82)**：將金鑰從主機發送到讀卡器內部的易失性儲存區
  2. **認證 (FF 86)**：讀卡器使用指定插槽中的金鑰，與卡片進行三次握手挑戰-應答協議

**Q: 不同扇區的 Trailer（最後一塊）有何風險？**
- **解答：** Sector Trailer 儲存了兩個金鑰和該扇區的「存取控制位元」
  - **風險 1：寫入失敗** - 可能導致整個扇區被鎖死
  - **風險 2：權限失控** - 存取控制位元被意外修改會導致數據塊無法訪問
- **教學建議：** 強烈要求學生在實驗中遠離 Trailer 塊的寫入操作

### **測試點**
- 成功讀到 Block 4（一般為可寫資料區第一塊）
- 能說明扇區/區塊關係

---

## **4. 實驗 3：MIFARE Classic 寫入與「小額錢包」**

### **實驗目標**
- 寫入自訂資料到指定區塊
- 用「讀→解析→改值→寫回」實作簡化錢包系統

### **程式碼實作**
```python
# exp3_mfc_wallet.py
from smartcard.System import readers
from smartcard.util import toHexString

LOAD_KEY = lambda slot, key: [0xFF, 0x82, 0x00, slot, 0x06] + key
AUTH = lambda block, key_type, slot: [0xFF, 0x86, 0x00, 0x00, 0x05, 0x01, 0x00, block, key_type, slot]
READ = lambda block: [0xFF, 0xB0, 0x00, block, 0x10]
WRITE = lambda block, data: [0xFF, 0xD6, 0x00, block, 0x10] + data

KEY_A = [0xFF] * 6
SLOT = 0x00
KEY_TYPE_A = 0x60
BLOCK = 4

def connect():
    conn = readers()[0].createConnection()
    conn.connect()
    return conn

def auth(conn, block=BLOCK):
    sw1, sw2 = conn.transmit(LOAD_KEY(SLOT, KEY_A))[1:]
    assert (sw1, sw2) == (0x90, 0x00), "載入金鑰失敗"
    sw1, sw2 = conn.transmit(AUTH(block, KEY_TYPE_A, SLOT))[1:]
    assert (sw1, sw2) == (0x90, 0x00), "認證失敗"

def read_balance(conn):
    data, sw1, sw2 = conn.transmit(READ(BLOCK))
    assert (sw1, sw2) == (0x90, 0x00), "讀取失敗"
    raw = bytes(data).decode('latin1', 'ignore').strip('\x00 ')
    if raw.startswith("BAL="):
        try:
            return int(raw.split("=")[1])
        except:
            return 0
    return 0

def write_balance(conn, bal):
    s = f"BAL={bal}".ljust(16, ' ')
    data = list(s.encode('latin1'))
    _, sw1, sw2 = conn.transmit(WRITE(BLOCK, data))
    assert (sw1, sw2) == (0x90, 0x00), f"寫入失敗: {sw1:02X} {sw2:02X}"

if __name__ == "__main__":
    conn = connect()
    auth(conn)
    b = read_balance(conn)
    print("Current:", b)
    b += 10
    write_balance(conn, b)
    print("After +10:", read_balance(conn))
```

### **教師解答與核心知識點**

**Q: 為何不直接用 Classic「Value Block」與加/減命令？**
- **解答：** 這是出於教學安全性、通用性和複雜度的綜合考慮
  1. **教學安全性與複雜度**：Value Block 操作需要精確的位元翻轉，操作不當易損壞數據
  2. **通用性與相容性**：READ 和 WRITE 命令是所有讀卡器都支持的基本操作
  3. **生物聯網應用啟發**：在物聯網應用中，記錄「使用次數」、「剩餘時長」等數據用普通的整數存儲已經足夠

### **測試點**
- 連續加值/扣值能正確顯示
- 數據持久化保存

---

## **5. 實驗 4：NTAG/Ultralight 讀寫與 NDEF**

### **實驗目標**
- 以「頁（4 bytes/page）」為單位讀寫
- 讀出 NDEF Text/URL 記錄
- 理解 TLV 結構與 NDEF 格式

### **程式碼實作**
```python
# exp4_t2_ndef_read.py
from smartcard.System import readers
from smartcard.util import toHexString

READ_PAGE = lambda page: [0xFF, 0xB0, 0x00, page, 0x04]  # 4 bytes/page

def read_pages(conn, start, length):
    buf = bytearray()
    for p in range(start, start + (length + 3) // 4):
        data, sw1, sw2 = conn.transmit(READ_PAGE(p))
        if (sw1, sw2) != (0x90, 0x00):
            break
        buf.extend(bytes(data))
    return bytes(buf)[:length]

def find_ndef(conn):
    # 掃描 4~48 頁（依卡容量可調）
    raw = bytearray()
    for p in range(4, 48):
        data, sw1, sw2 = conn.transmit(READ_PAGE(p))
        if (sw1, sw2) != (0x90, 0x00):
            break
        raw.extend(bytes(data))
    b = bytes(raw)
    i = 0
    while i < len(b):
        t = b[i]
        if t == 0x03:  # NDEF TLV
            l = b[i + 1]
            ndef = b[i + 2:i + 2 + l]
            return ndef
        elif t == 0xFE or t == 0x00:
            i += 1
        else:
            i += 2 + b[i + 1]
    return None

def parse_text(ndef):
    # 簡單解析 Text Record（TNF=Well-known 'T'）
    if not ndef or len(ndef) < 3:
        return None
    # 例：D1 01 xx 54 <len+lang+text>
    if ndef[0] & 0xD8 != 0xD1:
        return None  # MB+SR+TNF=1
    if ndef[3] != 0x54:
        return None  # 'T'
    payload_len = ndef[2]
    payload = ndef[4:4 + payload_len]
    if not payload:
        return None
    lang_len = payload[0] & 0x3F
    text = payload[1 + lang_len:].decode('utf-8', 'ignore')
    return text

r = readers()
conn = r[0].createConnection()
conn.connect()
ndef = find_ndef(conn)
if ndef:
    print("NDEF:", toHexString(list(ndef)))
    text = parse_text(ndef)
    print("TEXT:", text if text else "(非文字記錄)")
else:
    print("未找到 NDEF TLV，可能是空卡或為非 NDEF 格式。")
```

### **教師解答與核心知識點**

**Q: NDEF 解析過程的核心是什麼？**
- **解答：** 核心是理解 **TLV 結構**和 **NDEF 記錄格式**
  1. **TLV (Tag-Length-Value)**：
     - `03`：Tag，代表這是一個 NDEF 消息
     - `xx`：Length，代表後續 NDEF 數據的長度
     - `(數據)`：Value，即完整的 NDEF 消息
  2. **NDEF 記錄解析**：
     - 示例 `D1 01 0A 54 02 65 6E 48 65 6C 6C 6F`：
       - `D1`：標誌位（MB=1, ME=1, CF=0, SR=1, IL=0, TNF=001）
       - `01`：類型長度
       - `0A`：負載長度（10 位元組）
       - `54`：類型（'T'，代表 Text）
       - `02`：狀態位元組（IANA 語言碼長度為 2）
       - `65 6E`：語言碼（"en"）
       - `48 65 6C 6C 6F`：文本內容（"Hello"）

**生物聯網關聯：** NTAG 非常適合作為「物聯網世界的二維碼」，成本低、可讀寫。在生物樣本管理中，一個 NTAG 標籤可以存儲一個 NDEF URI 記錄，指向伺服器上該樣本的詳細信息頁面。

### **測試點**
- 能從 Page 4 開始掃描 TLV，解析文字或網址
- 理解頁結構與 NDEF 格式

---

## **6. 實驗 5：金鑰管理（高階操作）**

### **實驗目標**
- 在「實驗卡」上修改扇區金鑰
- 理解 MIFARE Classic 安全機制

### **操作步驟**
1. 讀出該扇區 Sector Trailer（最後一塊，含 Key A/存取位/Key B）
2. 設計新的 Key A（例如 A0 A1 A2 A3 A4 A5）與存取控制
3. **一次性**寫回 Sector Trailer
4. 之後以新 Key A 認證

### **教師指導要點**
- **安全第一：** 此實驗必須在**完全空白、可丟棄**的 MIFARE Classic 卡上進行
- **核心概念：** 讓學生理解「知識因子」（金鑰）在物聯網安全中的基礎地位
- **實作關鍵：**
  1. 讀取原 Sector Trailer 備份
  2. 計算新的存取控制位元組
  3. 構建新的 Trailer 數據
  4. **一次性**寫入，並立即用新金鑰驗證

---

## **7. 實驗 6：Flask 即時點名/資產盤點系統**

### **實驗目標**
- 建立 Web-based NFC 應用系統
- 實現多卡片輪流刷，列表即時更新

### **程式碼實作**
```python
# exp6_flask_demo.py
from flask import Flask, render_template_string
from threading import Thread
from time import sleep
from datetime import datetime

from smartcard.System import readers
from smartcard.util import toHexString

GET_UID = [0xFF, 0xCA, 0x00, 0x00, 0x00]
LOAD_KEY = lambda slot, key: [0xFF, 0x82, 0x00, slot, 0x06] + key
AUTH = lambda block, key_type, slot: [0xFF, 0x86, 0x00, 0x00, 0x05, 0x01, 0x00, block, key_type, slot]
READ = lambda block: [0xFF, 0xB0, 0x00, block, 0x10]

KEY_A = [0xFF] * 6
SLOT = 0x00
KEY_TYPE_A = 0x60
BLOCK = 4

app = Flask(__name__)
events = []  # 近況

PAGE = """
<!doctype html><meta charset="utf-8">
<title>ACR122U 刷卡監視</title>
<h1>刷卡事件（最近 20 筆）</h1>
<table border=1 cellpadding=6>
<tr><th>時間</th><th>UID</th><th>Balance(若有)</th></tr>
{% for e in events %}
<tr><td>{{e.ts}}</td><td>{{e.uid}}</td><td>{{e.bal}}</td></tr>
{% endfor %}
</table>
"""

def poller():
    r = readers()
    if not r:
        print("找不到讀卡機")
        return
    conn = r[0].createConnection()
    conn.connect()
    seen = None
    while True:
        try:
            data, sw1, sw2 = conn.transmit(GET_UID)
            if (sw1, sw2) == (0x90, 0x00):
                uid = toHexString(data)
                if uid != seen:
                    # 讀取 Block4 的 BAL=
                    conn.transmit(LOAD_KEY(SLOT, KEY_A))
                    conn.transmit(AUTH(BLOCK, KEY_TYPE_A, SLOT))
                    d, sw1, sw2 = conn.transmit(READ(BLOCK))
                    bal = ""
                    if (sw1, sw2) == (0x90, 0x00):
                        raw = bytes(d).decode('latin1', 'ignore').strip('\x00 ')
                        if raw.startswith("BAL="):
                            bal = raw.split("=")[1]
                    events.insert(0, dict(ts=datetime.now().strftime("%Y-%m-%d %H:%M:%S"), uid=uid, bal=bal))
                    del events[20:]
                    seen = uid
            else:
                seen = None
        except Exception:
            seen = None
        sleep(0.5)

@app.route("/")
def index():
    return render_template_string(PAGE, events=events)

if __name__ == "__main__":
    Thread(target=poller, daemon=True).start()
    app.run(host="0.0.0.0", port=5000, debug=False)
```

### **系統架構與生物聯網擴展指導**

**架構分析：**
- 這是一個典型的**邊緣計算**原型
- 讀卡器作為感知層設備，Python 程式負責邊緣的數據採集與初步處理

**生物聯網項目擴展建議：**
1. **數據持久化**：將刷卡事件存入 SQLite 或 MySQL 數據庫
2. **RESTful API**：將 Flask 改造成 API 服務端
3. **消息隊列與雲端集成**：將事件發送至 MQTT 主題或雲端 IoT 平台
4. **與生物設備聯動**：通過 GPIO 控制繼電器模組來模擬開啟實驗設備

---

## **8. 評分標準**

| 指標 | 達成描述 | 配分 |
|------|----------|------|
| **環境建置與基礎操作** | 正確安裝驅動與庫，穩定讀取 UID 與 ATR | 15 |
| **MIFARE Classic 讀寫** | 理解扇區與塊結構，成功完成認證、讀取、寫入及錢包功能 | 35 |
| **NTAG NDEF 解析** | 理解頁結構與 TLV，成功讀取並解析 NDEF Text/URL 記錄 | 20 |
| **系統集成與創新** | Flask Demo 運行穩定，界面清晰，並能提出有價值的生物聯網應用擴展思路 | 20 |
| **實驗報告與安全規範** | 報告結構完整，步驟清晰，對問題有深入反思，嚴格遵守操作安全 | 10 |

---

## **9. 附錄：ACR122U 常用 PC/SC APDU 速查表**

| 功能 | APDU 指令 | 說明 |
|------|-----------|------|
| 取 UID | `FF CA 00 00 00` | 取得卡片 UID |
| 載入金鑰 | `FF 82 00 <slot> 06 <6-byte-key>` | 載入金鑰到指定插槽 |
| 認證 | `FF 86 00 00 05 01 00 <block> <60\|61> <slot>` | 認證指定區塊 |
| 讀 Classic 區塊 | `FF B0 00 <block> 10` | 讀取 16 位元組區塊 |
| 寫 Classic 區塊 | `FF D6 00 <block> 10 <16-bytes>` | 寫入 16 位元組區塊 |
| 讀 Type 2 頁 | `FF B0 00 <page> 04` | 讀取 4 位元組頁 |
| 寫 Type 2 頁 | `FF D6 00 <page> 04 <4-bytes>` | 寫入 4 位元組頁 |

---

## **生物聯網應用場景總結**

1. **實驗設備門禁與使用追蹤**：MIFARE 卡作為設備使用權限憑證
2. **樣本管與試劑追蹤**：NTAG 標籤存儲樣本 ID、採集時間等信息
3. **溫室環境數據標記**：NFC 標籤用於地理位置數據綁定
4. **資產盤點與維護記錄**：貴重儀器配備 NFC 標籤記錄維護信息

---

**教師備註：**
- 本講義針對生物聯網系學生設計，重點強調物聯網應用場景
- 實驗難度由淺入深，適合 2-4 次實驗課完成
- 強調安全操作規範，避免卡片損壞和安全風險

---
