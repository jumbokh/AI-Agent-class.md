# 子專案一：異構模態資料的幾何對應建模與對齊理論

本子專案聚焦於 RTI（Reflectance Transformation Imaging）與 3D 掃描之間的空間幾何對齊問題，研究非同步影像源之間的幾何轉換關係、對應點匹配策略與配準算法的數學基礎。

---

## 🎯 研究目標

建立 RTI 與 3D 掃描資料之間的幾何對應理論模型與對齊演算法，處理異構模態在結構維度與空間表示上的不一致性問題。

---

## 🔧 技術流程與方法

### 1. 模態資料預處理與特徵提取

- RTI 紋理特徵提取：
![公式](https://latex.codecogs.com/png.image?f_%7BRTI%7D%28x%2C%20y%29%20%3D%20%5Cnabla%20I%28x%2C%20y%2C%20%5Ctheta%29)

- 3D 幾何特徵提取：
  - 表面法線向量：![公式](https://latex.codecogs.com/png.image?%5Cvec%7Bn%7D%28x%2C%20y%2C%20z%29)
  - 表面曲率：![公式](https://latex.codecogs.com/png.image?H), ![公式](https://latex.codecogs.com/png.image?K)

---

### 2. 建立共同幾何特徵空間

- 特徵距離函數：
![公式](https://latex.codecogs.com/png.image?d_%7Bij%7D%20%3D%20%5C%7Cf_%7BRTI%7D%28p_i%29%20-%20f_%7B3D%7D%28q_j%29%5C%7C)

---

### 3. 粗配準與對應點匹配

- 初始剛性轉換模型：
![公式](https://latex.codecogs.com/png.image?T%28p%29%20%3D%20R%20%5Ccdot%20p%20%2B%20t)
  其中 ![公式](https://latex.codecogs.com/png.image?R%20%5Cin%20SO%283%29)，![公式](https://latex.codecogs.com/png.image?t%20%5Cin%20%5Cmathbb%7BR%7D%5E3)

---

### 4. 精細對齊與幾何轉換建模

- 非剛性轉換模型（Thin Plate Spline）：
![公式](https://latex.codecogs.com/png.image?T%28p%29%20%3D%20A%20%5Ccdot%20p%20%2B%20%5Csum_%7Bi%3D1%7D%5EN%20w_i%20%5C%2C%20%5Cphi%28%5C%7Cp%20-%20c_i%5C%7C%29)
  其中 ![公式](https://latex.codecogs.com/png.image?%5Cphi%28r%29%20%3D%20r%5E2%20%5Clog%20r)

---

### 5. 對齊精度評估與理論分析

- 均方根誤差（RMSE）：
![公式](https://latex.codecogs.com/png.image?RMSE%20%3D%20%5Csqrt%7B%5Cfrac%7B1%7D%7Bn%7D%20%5Csum_%7Bi%3D1%7D%5En%20%5C%7CT%28p_i%29%20-%20q_i%5C%7C%5E2%7D)

- Hausdorff 距離：
![公式](https://latex.codecogs.com/png.image?d_H%28P%2C%20Q%29%20%3D%20%5Cmax%20%5Cleft%5C%7B%20%5Csup_%7Bp%20%5Cin%20P%7D%20%5Cinf_%7Bq%20%5Cin%20Q%7D%20%5C%7Cp%20-%20q%5C%7C%2C%20%5Csup_%7Bq%20%5Cin%20Q%7D%20%5Cinf_%7Bp%20%5Cin%20P%7D%20%5C%7Cq%20-%20p%5C%7C%20%5Cright%5C%7D)

---

## ✅ 小結

本子專案通過一系列幾何建模與特徵對齊機制，建立異構資料間的精確空間對應關係，為後續特徵融合與 AI 修復模型提供理論基礎與數據支撐。
