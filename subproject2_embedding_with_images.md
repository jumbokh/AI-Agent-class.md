# 子專案二：跨模態影像特徵融合與高維嵌入結構研究

本子專案致力於建立一個能夠統一表示 RTI 與 3D 掃描資料的高維特徵嵌入空間，以支持多模態資料之間的結構理解與語義推理。嵌入空間應具備可分性、幾何保持性與穩定性等特性。

---

## 🔧 技術流程與方法

### 1. 特徵提取與預處理

- RTI：提取紋理梯度、光照變化特徵
- 3D：提取表面法線、曲率、深度圖
- 模型工具：ResNet、PointNet++、MeshCNN

### 2. 聯合嵌入空間構建

- 使用聯合編碼器或對比學習方式建構共享嵌入空間
- 損失函數：
  - Triplet loss:
![公式](https://latex.codecogs.com/png.image?L%20%3D%20%5Cmax%280%2C%20%5C%7Cf_a%20-%20f_p%5C%7C%5E2%20-%20%5C%7Cf_a%20-%20f_n%5C%7C%5E2%20%2B%20%5Calpha%29)

  - Contrastive loss:
![公式](https://latex.codecogs.com/png.image?L%20%3D%20y%20%5Ccdot%20%5C%7Cf_1%20-%20f_2%5C%7C%5E2%20%2B%20%281%20-%20y%29%20%5Ccdot%20%5Cmax%280%2C%20m%20-%20%5C%7Cf_1%20-%20f_2%5C%7C%29%5E2)

### 3. 嵌入空間評估與分析

- 可視化：使用 t-SNE、UMAP 檢查流形保持性
- 可分性指標：Silhouette Score、Davies-Bouldin Index
- 穩定性：不同初始化與訓練條件下嵌入空間的變異性分析

### 4. 下游應用測試

- 文物分類、缺陷預測、多模態檢索
- 評估指標：
  - Accuracy, F1-score
  - 檢索準確率：![公式](https://latex.codecogs.com/png.image?Recall%40K)
---

## ✅ 小結

本子專案強調理論推導與嵌入結構品質分析，並為後續修復模型提供統一、多模態的語義表示空間。
