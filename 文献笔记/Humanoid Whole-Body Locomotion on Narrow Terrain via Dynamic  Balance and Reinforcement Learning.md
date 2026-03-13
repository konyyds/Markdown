# Humanoid Whole-Body Locomotion on Narrow Terrain via Dynamic  Balance and Reinforcement Learning - 总结笔记

## 第1章：引言 (Introduction)

### 研究背景
- 现有的人形机器人运动算法在极端环境（尤其是缺乏外部感知时）仍表现不佳
- 当前方法主要依赖基于步态（gait-based）或感知条件（perception-condition）的奖励，缺乏处理不可观测障碍和突然失衡的有效机制

### 主要贡献
1. 将 **零力矩点ZMP**作为新型奖励函数集成到RL框架中
2. 构建包含新技术（**奖励向量化、角动量正则化、乘性动作噪声**）的动态平衡人形运动(DBHL)框架，仅使用本体感知，无需视觉或LiDAR
3. 通过仿真和真实世界实验进行评估，1.8 米高的仿人机器人在没有视觉的情况下稳健地走过 25 厘米宽的桥。

## 第3章：方法 (Method)

### 整体框架
非对称Actor-Critic框架包含两个独立的神经网络，它们接收不同的输入：

#### 1. Actor 网络（策略网络 $π$）
- **输入**：仅包含**本体感知信息 $s_t^{prop}$** 和 **速度命令$c_t$**。
- **观察空间 $o_t$ (279维)**：
  - **速度命令 $c_t$**： `[v̂_x,t, v̂_y,t, ω̂_yaw,t]` (3维)
  - **本体感知历史 $s_t^{prop}$**：包含过去4个时间步的信息 (276维)
    - 关节位置 `q` (21 DoF)
    - 关节速度 `q̇`
    - 基座角速度 `ω`
    - 基座投影重力向量 `g`
    - 上一时刻的动作 `a`

#### 2. Critic 网络（价值网络 $V$）
- **输入**：包含Actor的全部观察信息，外加仅在仿真中可用的**privilege信息 $s_t^{priv}$**和**环境地形信息$s_t^{hf}$**。
- **观察空间 $s_t$**：
  - **Actor的所有观察 $o_t$** (279维)
  - **privilege信息 $s_t^{priv}$** (70维)：例如真实的线速度、基座高度、脚部接触状态、随机化的PD参数和质量等。
  - **周围高度场 $s_t^{hf}$** (187维)：在机器人周围一个1.6m x 1.0m的区域内采样得到的地形高程信息。

![Overview](.\images\ScreenShot_2025-10-30_112402_356.png)

### 3.1 基于ZMP的动态平衡（ZMP-based Dynamic Balance）
#### ZMP（Zero Moment Point）奖励函数
**最小化零力矩线（ZML）与近似支撑多边形中心（CSP）之间的水平距离d**

- **ZML**：由不同高度层面的零力矩点（ZMP）形成的一条线  
- **CSP**：通过支撑脚的中点位置近似计算得出  
- **直观原理**：促使ZMP保持在支撑多边形内部，以维持动态平衡。

![r_zmp](.\images\ScreenShot_2025-10-30_113038_368.png)
![fig3](.\images\ScreenShot_2025-10-30_113617_337.png)

#### 支撑多边形中心（CSP）由支撑脚中心近似计算：
![pcsp](.\images\ScreenShot_2025-10-30_120822_865.png) 

#### ZMP计算（推导见附1）
- 基于牛顿-欧拉方程推导ZMP位置：
![Formula 4&5](.\images\ScreenShot_2025-10-30_114642_734.png)

### 3.2 全身运动 (Whole-Body Locomotion)

#### 角动量正则化（Angular Momentum Regularization）
- 目的：鼓励上肢运动来抵消摆动双腿产生的角动量，降低整体角动量
![ram](.\images\ScreenShot_2025-10-30_122520_958.png) 


#### 乘性动作噪声注入（Multiplicative Action Noise Injection）
- 目的：约束上身关节运动范围。通过乘性噪声，大动作受到更强扰动，鼓励策略学习更保守的上身运动
![at](.\images\ScreenShot_2025-10-30_122801_241.png) 

### 3.3 策略学习细节

#### 非对称Actor-Critic框架
- **Actor观察**(279维)：速度命令 + 4步历史的本体感知信息
- **Critic观察**：Actor观察 + 特权信息(70维) + 周围高度场(187维)

这种非对称设计的根本目的是为了解决仿真与现实之间的差距：

1.  **保证策略的可转移性**：
    - Actor只使用在真实机器人上可以直接、可靠地通过IMU、编码器等传感器测量的本体感知信息。
    - 这确保了在仿真中训练出的策略能够直接部署到真实的硬件上，而无需依赖那些在现实中不可靠或根本不存在的传感器（如精确的地形高程图）。

2.  **提升训练效率与稳定性**：
    - Critic利用丰富的特权信息和全局地形信息，能够做出更准确、更明智的价值评估。
    - 更准确的价值估计为Actor的策略更新提供了更高质量的学习信号，从而加速训练收敛并提升最终策略的性能。

#### 奖励与价值向量化（与标量奖励比较见附2）
- 将不同奖励组合为向量，分别学习对应的价值函数
- 总价值函数：
  \[
  V^{\text{total}}(s_t) = \sum_{i=1}^{\#\text{Reward}} V^i(s_t)
  \]
- 价值函数损失：
  \[
  L_{\text{value}} = \sum_{i=1}^{\#\text{Reward}} \mathbb{E}\Big[\big|\big|r_t^i + \gamma V^i(s_{t+1}) - V^i(s_t)\big|\big|_2^2\Big]
  \]

#### 对称性正则化
- 利用机器人运动的对称性提升样本效率和步态协调性
- 对称损失：
$$ L_{\text{symm}} = \mathbb{E}\left[\left\|\bm{V}(G(\bm{s}_t)) - \bm{V}(\bm{s}_t)\right\|_2^2 + \left\|\bm{\pi}(G(\bm{o}_t)) - G(\bm{\pi}(\bm{o}_t))\right\|_2^2\right] $$

#### 域随机化
- 实现零样本sim-to-real转移
- 随机化参数：外部推力、动作延迟、PD增益、摩擦、连杆质量、负载质量等



### 附1：ZMP公式4和5的推导过程

假设地面反作用力是作用于机器人系统的唯一外力
- 公式1 该地面反作用力相对于世界坐标系原点的力矩: $$\bm{\tau} = \bm{p}_{\text{zmp}} \times \bm{f} + \bm{\tau}_p$$
其中，$\bm{p}_{\text{zmp}}$表示ZMP的位置，$\bm{f}$ 则代表地面反作用力, $\bm{\tau}_p$ 表示关于ZMP的力矩

- 公式2 牛顿-欧拉方程：描述了动量和角动量的变化率 
  $$
  \begin{cases}
  \dot{\bm{\mathcal{P}}} = M\bm{g} + \bm{f} \\
  \dot{\bm{\mathcal{L}}} = \bm{p}_{\text{CoM}} \times M\bm{g} + \bm{\tau}
  \end{cases}
  $$
- 公式3 ZMP点处水平力矩为零: $$\tau_{p,x} = \tau_{p,y} = 0$$

推导步骤：
1. 从公式2.1解出地面反力 $\bm{f}$:
   $$
   \bm{f} = \dot{\bm{\mathcal{P}}} - M\bm{g}
   $$

2. 将 $\bm{f}$ 代入公式1:
   $$
   \bm{\tau} = \bm{p}_{\text{zmp}} \times (\dot{\bm{\mathcal{P}}} - M\bm{g}) + \bm{\tau}_p
   $$

3. 将 $\bm{\tau}$ 代入公式2.2:
   $$
   \dot{\bm{\mathcal{L}}} = \bm{p}_{\text{CoM}} \times M\bm{g} + \bm{p}_{\text{zmp}} \times (\dot{\bm{\mathcal{P}}} - M\bm{g}) + \bm{\tau}_p
   $$
   展开后：
   $$
   \dot{\bm{\mathcal{L}}} = \bm{p}_{\text{CoM}} \times M\bm{g} + \bm{p}_{\text{zmp}} \times \dot{\bm{\mathcal{P}}} - \bm{p}_{\text{zmp}} \times M\bm{g} + \bm{\tau}_p
   $$
   重组项：
   $$
   \dot{\bm{\mathcal{L}}} = (\bm{p}_{\text{CoM}} - \bm{p}_{\text{zmp}}) \times M\bm{g} + \bm{p}_{\text{zmp}} \times \dot{\bm{\mathcal{P}}} + \bm{\tau}_p
   $$

4. 考虑水平分量（x和y）：  
   由于公式3中 $\tau_{p,x} = 0$ 和 $\tau_{p,y} = 0$，我们只关心 $\dot{\bm{\mathcal{L}}}$ 的x和y分量。假设世界坐标系中z轴垂直向上，重力加速度 $\bm{g} = [0, 0, -g]^\top$。

   - 计算 $(\bm{p}_{\text{CoM}} - \bm{p}_{\text{zmp}}) \times M\bm{g}$:
     $$
     (\bm{p}_{\text{CoM}} - \bm{p}_{\text{zmp}}) \times M\bm{g} = M \begin{bmatrix} -g (p_{\text{CoM},y} - p_{\text{zmp},y}) \\ g (p_{\text{CoM},x} - p_{\text{zmp},x}) \\ 0 \end{bmatrix}
     $$

   - 计算 $\bm{p}_{\text{zmp}} \times \dot{\bm{\mathcal{P}}}$:
     $$
     \bm{p}_{\text{zmp}} \times \dot{\bm{\mathcal{P}}} = \begin{bmatrix} p_{\text{zmp},y} \dot{\mathcal{P}}_z - p_{\text{zmp},z} \dot{\mathcal{P}}_y \\ p_{\text{zmp},z} \dot{\mathcal{P}}_x - p_{\text{zmp},x} \dot{\mathcal{P}}_z \\ p_{\text{zmp},x} \dot{\mathcal{P}}_y - p_{\text{zmp},y} \dot{\mathcal{P}}_x \end{bmatrix}
     $$

   - *代入力矩方程*：
     - 对于 $\dot{\mathcal{L}}_x$（x分量）：
       $$
       \dot{\mathcal{L}}_x = -M g (p_{\text{CoM},y} - p_{\text{zmp},y}) + p_{\text{zmp},y} \dot{\mathcal{P}}_z - p_{\text{zmp},z} \dot{\mathcal{P}}_y
       $$
       重组为：
       $$
       \dot{\mathcal{L}}_x = p_{\text{zmp},y} (M g + \dot{\mathcal{P}}_z) - M g p_{\text{CoM},y} - p_{\text{zmp},z} \dot{\mathcal{P}}_y
       $$
       解得：
       $$
       p_{\text{zmp},y} (M g + \dot{\mathcal{P}}_z) = M g p_{\text{CoM},y} + p_{\text{zmp},z} \dot{\mathcal{P}}_y + \dot{\mathcal{L}}_x
       $$
       即公式5：
       $$
       p_{\text{zmp},y} = \frac{M g p_{\text{CoM},y} + p_{\text{zmp},z} \dot{\mathcal{P}}_y + \dot{\mathcal{L}}_x}{M g + \dot{\mathcal{P}}_z}
       $$

     - 对于 $\dot{\mathcal{L}}_y$（y分量）：
       $$
       \dot{\mathcal{L}}_y = M g (p_{\text{CoM},x} - p_{\text{zmp},x}) + p_{\text{zmp},z} \dot{\mathcal{P}}_x - p_{\text{zmp},x} \dot{\mathcal{P}}_z
       $$
       重组为：
       $$
       \dot{\mathcal{L}}_y = M g p_{\text{CoM},x} + p_{\text{zmp},z} \dot{\mathcal{P}}_x - p_{\text{zmp},x} (M g + \dot{\mathcal{P}}_z)
       $$
       解得：
       $$
       p_{\text{zmp},x} (M g + \dot{\mathcal{P}}_z) = M g p_{\text{CoM},x} + p_{\text{zmp},z} \dot{\mathcal{P}}_x - \dot{\mathcal{L}}_y
       $$
       即公式4：
       $$
       p_{\text{zmp},x} = \frac{M g p_{\text{CoM},x} + p_{\text{zmp},z} \dot{\mathcal{P}}_x - \dot{\mathcal{L}}_y}{M g + \dot{\mathcal{P}}_z}
       $$

### 附2：奖励与价值向量化的详细说明

这部分解决的是强化学习中的一个常见问题：**当奖励函数由多个不同目标的项相加时，如何更有效地进行学习。**

1. 传统方法的缺陷

在传统的强化学习中，通常的做法是将所有奖励项（如速度跟踪、ZMP平衡、角动量正则化等）加权求和，合并成一个单一的标量奖励 \( R_{total} \)：
\[
R_{total} = w_1 r_1 + w_2 r_2 + w_3 r_3 + ...
\]
然后，再为这个标量奖励学习一个单一的价值函数 \( V_{total} \)。

这种方法的缺点是：
- 信用分配问题：当总奖励发生变化时，很难判断是哪个奖励项导致的。价值函数无法精确感知每个独立奖励项的变化。
- 小奖励项被淹没：一些重要但数值较小的奖励项（例如平衡相关的奖励）可能被数值较大但目标简单的奖励项（例如速度跟踪）所“淹没”，导致策略无法有效学习那些精细但关键的行为。

2. 本文的解决方案：奖励与价值向量化

本文提出了一种“向量化”的方法，不再将奖励合并为一个标量，而是将其保持为一个奖励向量：
\[
\bm{r}_t = [r_t^1, r_t^2, r_t^3, ..., r_t^{\#Reward}]
\]

相应地，价值函数也不再是单一输出，而是一个具有多个输出头的神经网络，每个头负责学习一个奖励项对应的价值函数：
\[
\bm{V}(s_t) = [V^1(s_t), V^2(s_t), V^3(s_t), ..., V^{\#Reward}(s_t)]
\]
每个价值函数 \( V^i \) 只用于预测其对应奖励项 \( r^i \) 的累积未来回报。

3. 具体实现方式

- **独立学习**：每个价值函数 \( V^i \) 使用其自己的时序差分目标进行独立训练，损失函数为：
  \[
  L_{\text{value}} = \sum_{i=1}^{\#\text{Reward}} \mathbb{E}\Big[\big|\big|r_t^i + \gamma V^i(s_{t+1}) - V^i(s_t)\big|\big|_2^2\Big]
  \]
  这确保了每个 \( V^i \) 都能准确地学习到其对应奖励项的动态变化。

- **协同工作**：在需要计算总体优势函数（用于策略更新）时，将这些独立的价值函数临时加总：
  \[
  V^{\text{total}}(s_t) = \sum_{i=1}^{\#\text{Reward}} V^i(s_t)
  \]
  然后，用这个加总后的价值函数来计算优势 \( A^{\text{total}} \)，并用于更新策略。

4. 核心优势

- **精准的信用分配**：策略能够清晰地了解到哪种行为影响了哪个特定的奖励项，从而进行更精确的优化。
- **保护小奖励项**：即使某个奖励项的数值很小，因为它拥有独立的价值函数，其变化也能被策略敏锐地捕捉和学习到，不会被其他奖励项干扰。
- **提升学习效率与稳定性**：通过更清晰的学习信号，策略能够更快地收敛，并且在复杂任务中表现更稳定。







