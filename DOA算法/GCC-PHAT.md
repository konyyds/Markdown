# GCC-PHAT 算法介绍
## 1. 背景与原理
GCC-PHAT（Generalized Cross-Correlation with Phase Transform）是一种广泛应用于信号处理领域的时延估计算法，尤其在声学中用于声源定位或估计信号到达不同传感器的时间差（TDOA, Time Difference of Arrival）。它通过增强传统互相关方法在噪声和混响环境中的鲁棒性，成为时延估计的重要工具。
### 1.1 互相关函数
互相关函数是时延估计的基础。给定两个信号 $x_1(t)$ 和 $x_2(t)$，其互相关函数 $R_{x_1 x_2}(\tau)$ 定义为：
$$ R_{x_1 x_2}(\tau) = \int_{-\infty}^{\infty} x_1(t) x_2(t + \tau) dt $$
峰值位置 $\tau_{\text{max}}$ 通常对应两信号间的时延。然而，在噪声或多径传播的影响下，峰值可能变得模糊或出现多个伪峰。
### 1.2 广义互相关（GCC）
为了克服传统互相关的局限性，Knapp 和 Carter 于 1976 年提出了广义互相关（GCC）方法。GCC 在频域中通过对互功率谱加权来提高时延估计的精度。其核心思想是引入预滤波器，根据信号特性和应用场景选择合适的加权函数。
### 1.3 PHAT加权
PHAT（Phase Transform）是GCC的一种特殊加权方式，通过将互功率谱的幅度归一化为1，仅保留相位信息。其公式为：
$$ R_{\text{GCC-PHAT}}(\tau) = \int_{-\infty}^{\infty} \frac{X_1(f) X_2^*(f)}{|X_1(f) X_2^*(f)|} e^{j 2 \pi f \tau} df $$
其中：

$X_1(f)$ 和 $X_2(f)$ 分别是 $x_1(t)$ 和 $x_2(t)$ 的傅里叶变换；
$X_2^*(f)$ 是 $X_2(f)$ 的共轭；
$|X_1(f) X_2^*(f)|$ 是互功率谱的幅度。

PHAT加权消除了幅度对时延估计的影响，使其在混响或幅度失真的环境中表现更优。
## 2. 算法实现
### 2.1 离散时间实现步骤
在实际应用中，信号通常是离散的，因此GCC-PHAT使用离散傅里叶变换（DFT）实现。以下是具体步骤：

1. 信号预处理：对输入信号 $x_1[n]$ 和 $x_2[n]$ 进行预处理（如去直流、归一化）。
2. 傅里叶变换：计算 $X_1[k] = \text{DFT}(x_1[n])$ 和 $X_2[k] = \text{DFT}(x_2[n])$。
3. 互功率谱计算：计算 $G[k] = X_1[k] \cdot X_2^*[k]$。
4. PHAT加权：归一化互功率谱，$G_{\text{PHAT}}[k] = \frac{G[k]}{|G[k]|}$。
5. 逆傅里叶变换：对 $G_{\text{PHAT}}[k]$ 进行逆DFT，得到 $r_{\text{GCC-PHAT}}[n]$。
峰值检测：找到 $r_{\text{GCC-PHAT}}[n]$ 的最大值位置 $n_{\text{max}}$，时延为 $\tau = \frac{n_{\text{max}}}{f_s}$（$f_s$ 为采样频率）。

### 2.2 Matlab代码示例
以下是一个简化的GCC-PHAT实现：
```matlab
function [delay] = GccPhat(x1, x2, fs, N)
    % 傅里叶变换至频域
    x1_fft = fft(x1, N);
    x2_fft = fft(x2, N);
    
    % 计算互功率谱
    G = x1_fft .* conj(x2_fft);
    
    % PHAT加权（避免除以0）
    w = 1 ./ (abs(G) + eps);
    Gw = G .* w;
    
    % 逆傅里叶变换
    R12 = ifft(Gw);
    
    % 零频平移
    R12_shift = fftshift(R12);
    
    % 找峰值
    [~, idx] = max(abs(R12_shift));
    sIndex = -floor(N/2) : floor(N/2)-1;
    delay = -sIndex(idx) / fs;
end
```
注意：为避免除以0，加入微小值 eps。


## 3. 总结
GCC-PHAT通过相位变换加权改进了传统互相关方法，在噪声和混响环境中表现出色。其核心在于利用相位信息估计时延，适用于宽带信号和声学应用。然而，在强多径或窄带信号场景下，可能需要结合其他技术优化性能。