使用說明（簡版）
1) 安裝相依
pip install opencv-contrib-python numpy


必須是 opencv-contrib-python，因為需要 cv2.aruco。

2) 產生四邊形設定（由實體邊長）
python aruco_measure_cli.py init \
  --name MyQuad \
  --L1 100 --L2 80 --L3 100 --L4 80 \
  --D 128.06 \
  -o myquad.json


對於矩形 W×H，可用：L1=L3=H, L2=L4=W, D=sqrt(W^2+H^2)。

3) 量測（算出「每像素對應的實體長度」）
python aruco_measure_cli.py measure \
  --image your_photo.jpg \
  --quad myquad.json \
  --out warped.png


會在終端印出 pixel_length = ...。

若偵測不到 4 個 ArUco（DICT_4X4_50），會回傳 100.0 代表失敗；同時不輸出 warped.png。

4) 只做透視變換（除錯用）
python aruco_measure_cli.py warp \
  --image your_photo.jpg \
  --quad myquad.json \
  --out warped.png

Notebook 範例怎麼跑

開啟 ArUco_Measure_Demo.ipynb。

先執行第一段安裝（若環境尚未安裝）。

在「Define the physical quad」小節中填入你的 L1..L4, D。

把 image_path 改成你的影像路徑，執行後會輸出：

pixel_length = ...

以及（若成功）將 warped_result.png 存到 /mnt/data/。

與 Android 版對應關係

init 子命令 = 原「解析程式」：由 
𝐿
1
∼
𝐿
4
,
𝐷
L
1
	​

∼L
4
	​

,D 求解實體四頂點 (dpx, dpy) 並存成設定。

measure/warp 子命令 = 原「量測程式」：偵測 4 個 ArUco → 外接四邊形 → 透視變換 → 計算四邊長像素距離，與實體邊長比值得到 pixel_length。
