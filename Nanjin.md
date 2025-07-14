Title: A Sub-Meter BLE-Based Indoor Positioning System with RSSI Kalman Filtering and Dynamic Path Loss Modeling

Authors:
Junlin Zhong, Jinghuang Chen, Jingyi Pan, Jiaxin Wu, Cong Gao
The School of Big Data, Fuzhou University of International Studies and Trade, Fuzhou, China
Emails: [individual emails]

Kunta Hsieh* (Corresponding Author)
The School of Big Data, Fuzhou University of International Studies and Trade, Fuzhou, China
Email: xiekunda@fzfu.edu.cn

Abstract:
This paper proposes a low-cost, sub-meter indoor positioning system utilizing Bluetooth Low Energy (BLE) iBeacons and Raspberry Pi devices. To overcome RSSI instability in indoor environments, a Kalman filter is applied to suppress signal noise, followed by dynamic calibration of a logarithmic path-loss model using environment-specific parameters. Final positioning is achieved through an inverse-distance weighted centroid algorithm. The system was validated in a 5 m × 5 m grid environment with 256 reference points, achieving a mean localization error of 0.28 m and reducing RSSI variance by 73.6%. Supporting real-time updates at 1 Hz and visual output, the system demonstrates strong potential for scalable, accurate indoor positioning in IoT applications such as smart buildings, elderly care, and asset tracking.
Keywords: Bluetooth Low Energy (BLE), Indoor Positioning, Kalman Filter, RSSI Filtering, Path Loss Model, Raspberry Pi, Smart Environments

Introduction
Indoor positioning is critical for various IoT applications, particularly where GPS is ineffective due to signal blockage and multipath interference. Technologies such as UWB, Wi-Fi, and RFID each have limitations in cost, accuracy, or applicability. BLE is a promising solution due to its accessibility and low cost; however, RSSI-based distance estimation is often unstable in indoor environments. This study proposes a BLE-based indoor positioning system integrating Kalman filtering and a dynamically optimized path-loss model. By employing Raspberry Pi devices and inverse-distance weighted centroid positioning, the system achieves real-time, sub-meter accuracy with minimal hardware requirements.

Related Work
2.1 Overview of Indoor Positioning Technologies
Indoor positioning methods are broadly categorized into signal-based and physical-property-based systems. Signal-based methods include Wi-Fi fingerprinting, UWB ranging, and ZigBee localization, each with trade-offs in cost, precision, and deployment complexity. Physical-property-based techniques, such as infrared, ultrasonic, and geomagnetic positioning, offer varying degrees of accuracy and robustness depending on environmental conditions.

2.2 BLE-Based Positioning
BLE-based positioning utilizes RSSI values derived from periodic beacon signals. The distance between a transmitter and receiver can be estimated using a logarithmic path-loss model, with parameters specific to the environment. While trilateration is commonly employed, its accuracy suffers due to RSSI noise. Kalman filtering and parameter optimization have been proposed to enhance precision by reducing signal variance and adapting to local conditions.

Methodology
3.1 Kalman Filtering
To mitigate RSSI instability caused by multipath and NLOS effects, a discrete-time Kalman filter is implemented. The state-space model includes a process noise component (Q = 0.1) and a measurement noise component (R = 7), representing environmental interference. The filter iteratively updates state estimates, reducing RSSI standard deviation and enhancing signal reliability.

3.2 Path Loss Model Optimization
Using 256 test points in the experiment area, an overdetermined system of RSSI-distance pairs is formulated. The optimal parameters A (signal strength at 1 meter) and n (path loss exponent) are determined via the least squares method, forming an environment-specific path-loss model.

3.3 Position Estimation
Three fixed BLE beacons are deployed at known coordinates. Filtered RSSI values are converted to distances, and the receiver's position is estimated using an inverse-distance weighted centroid algorithm. This method offers a balance between computational simplicity and accuracy.

Experimental Design
Experiments were conducted in a 5m × 5m grid with 0.3m spacing, yielding 256 test nodes. At each node, the Raspberry Pi device collected RSSI data at 1 Hz for 1-3 minutes. Kalman filtering and distance modeling were applied to compute the receiver's estimated position. The entire process, from signal acquisition to visualization, was streamlined into an automated workflow.

Results
The system achieved a mean localization error of 0.28 meters. Kalman filtering reduced RSSI standard deviation by 73.6%, significantly improving stability. Compared to traditional trilateration, the proposed inverse-distance weighted centroid method achieved a 39% accuracy improvement. Visualizations and data logs confirm the model's reliability and reproducibility.

Conclusion and Future Work
This study demonstrates the feasibility of sub-meter indoor positioning using BLE and Raspberry Pi. By integrating Kalman filtering and environment-specific path-loss modeling, the system achieves reliable performance in open indoor environments. Future work will address multi-floor and dynamic scenarios, explore sensor fusion with IMU and UWB, and apply deep learning models such as LSTM and CNN for further improvements.

Acknowledgment:
This research is supported by [Include funding source if available].

References:
[1] Shih, S. T., Hsieh, K., & Chen, P. Y. (2006). An Improvement Approach of Indoor Location Sensing Using Active RFID. In Proceedings of the First International Conference on Innovative Computing, Information and Control (ICICIC).
[2] Alsmadi, L., Kong, X., & Sandrasegaran, K. (2018). Improve Indoor Positioning Accuracy Using Filtered RSSI and Beacon Weight Approach in iBeacon Network. In 19th International Symposium on Communications and Information Technologies (ISCIT).
[3] Ni, L. M., Liu, Y., Lau, Y. C., et al. (2004). LANDMARC: Indoor Location Sensing Using Active RFID. Wireless Networks, 10(6), 701-710.
[4] Peng, D., Peng, L., & Yang, Y. (2021). Optimization of Indoor Bluetooth Ranging Model Based on RSSI. Frontiers in Signal Processing, 1, 45-53.
[5] Wang, W., Liu, X., Li, M., et al. (2019). Optimizing Node Localization in Wireless Sensor Networks Based on Received Signal Strength Indicator. IEEE Access, 7, 125758-125768.
[6] Chen, P. Y. (2007). Application of ZigBee Technology in Wireless Location Tracking System. Master’s Thesis, Zhengxiu University of Science and Technology.
[7] Yang, J. (2024). Research on Deployment Strategies and Positioning Algorithms for Low-Power Bluetooth Anchor Nodes for Indoor Localization. Harbin Engineering University.
