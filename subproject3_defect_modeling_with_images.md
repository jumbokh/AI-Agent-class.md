# 子專案三：多尺度缺陷識別與損壞模式建模

本子專案旨在建立一套能夠處理文物資料於多個空間尺度上進行缺陷辨識與損壞區域建模的技術體系，並透過深度學習、注意力機制與結構演化理論，進一步進行缺陷分割與樣式理解。

---

## 🔧 技術流程與方法

### 1. 多尺度影像與結構輸入建構
- 微觀：紋理細節、表面裂痕
- 中觀：區域剝落、碎裂
- 巨觀：形態歪斜、結構損毀
- 預處理：金字塔建構、空間對齊、ROI 區域提取

### 2. 缺陷分割模型構建
- 模型架構：使用多尺度卷積（例如 DeepLabV3+）、U-Net with Attention 模型
- 損失函數設計：
  - Dice Loss:
![公式](https://latex.codecogs.com/png.image?L_%7BDice%7D%20%3D%201%20-%20%5Cfrac%7B2%7CA%20%5Ccap%20B%7C%7D%7B%7CA%7C%20%2B%20%7CB%7C%7D)

  - Focal Loss:
![公式](https://latex.codecogs.com/png.image?L_%7BFocal%7D%20%3D%20-%5Calpha_t%20%281%20-%20p_t%29%5E%5Cgamma%20%5Clog%28p_t%29)

### 3. 損壞樣式建模與演化推理
- 利用結構圖（Graph）表示損壞組成單元與空間關係
- 樣式變異建模方法：
  - 有向圖轉移模型（Markov Chain）
  - 時序變化關係 $S_{t+1} = f(S_t, \Delta t)$

- 將損壞分為類別（裂、斷、蝕、翹），利用嵌入學習建構相似度圖嵌入空間：
![公式](https://latex.codecogs.com/png.image?L_%7Bgraph%7D%20%3D%20%5Csum_%7Bi%2Cj%7D%20A_%7Bij%7D%20%5Ccdot%20%5C%7Cf_i%20-%20f_j%5C%7C%5E2)

### 4. 評估指標
- 分割準確度（IoU、mIoU）
- 結構保持性（Topological Consistency）
- 分類精度與推理路徑準確率

---

## ✅ 小結
透過本子專案的推進，將建立一套結合影像理解與結構演化推理的缺陷識別體系，並為後續的 AI 修復建模提供語義級的損壞模板與推理基礎。
