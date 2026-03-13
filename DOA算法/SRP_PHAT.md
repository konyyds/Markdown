SRP-PHAT（Steered Response Power with Phase Transform）是一种**基于麦克风阵列的声源定位算法**，以其**在混响和噪声环境下的鲁棒性**被广泛应用于声学成像、语音识别、人机交互等领域。

---

## 一、传统 SRP-PHAT 算法简介

### 核心思想：

1. **通过假设声源位置**，计算所有麦克风对的**GCC-PHAT函数之和**；
2. 在声源空间中搜索最大值来估计声源位置。

### 主要步骤：

* **信号预处理**：分帧、加窗（如汉明窗）、FFT。
* **互相关计算**：对所有麦克风对求广义互相关（GCC）并加上PHAT加权。
* **方向搜索**：遍历空间点 $q$，找到对应最大 SRP-PHAT 值的位置。

### 数学表达：

* GCC-PHAT函数：

  $$
  R_{ml}(\tau) = \frac{1}{K} \sum_{n=0}^{K-1} \frac{X_m(n) X_l^*(n)}{|X_m(n) X_l^*(n)|} e^{j \omega_n \tau}
  $$
* SRP值计算：

  $$
  \hat{P}(q) = \sum_{m,l} R_{ml}[\tau_{ml}(q)]
  $$
* 定位估计：

  $$
  \hat{q} = \arg \max_q \hat{P}(q)
  $$

