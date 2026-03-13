# Balance Control of a Novel Wheel-legged Robot: Design and  Experiments

## II.B Simplification and Assumption
**手推车倒立摆 (Inverted Pendulum on a Cart, IPC)** 的动力学方程是如何建立的。这个系统的建模通常用 **拉格朗日方法 (Lagrangian Method)** 来推导。

---

## 1. 系统参数定义

* 小车（cart）质量：$M$
* 摆杆（pendulum）质量：$m$
* 摆杆长度：$l$（从转动点到质心的长度）
* 水平方向小车位置：$x$
* 摆杆与竖直方向的夹角：$\theta$（逆时针为正）
* 控制力：$u$（作用在小车上的水平推力）

---

## 2. 质心坐标

摆杆质心的位置坐标为：

$$
X = x + l \sin \theta
$$

$$
Y = l \cos \theta
$$

---

## 3. 速度

取时间导数，得到质心速度分量：

$$
\dot{X} = \dot{x} + l \cos \theta \, \dot{\theta}
$$

$$
\dot{Y} = -l \sin \theta \, \dot{\theta}
$$

---

## 4. 动能 (Kinetic Energy)

小车 + 摆杆的动能：

$$
T = \frac{1}{2}M \dot{x}^2 + \frac{1}{2}m\left(\dot{X}^2 + \dot{Y}^2\right)
$$

代入 $\dot{X}, \dot{Y}$：

$$
T = \frac{1}{2}M \dot{x}^2 + \frac{1}{2}m\left[ (\dot{x} + l\cos\theta \dot{\theta})^2 + (-l\sin\theta \dot{\theta})^2 \right]
$$

化简：

$$
T = \frac{1}{2}(M+m)\dot{x}^2 + ml\cos\theta \dot{x}\dot{\theta} + \frac{1}{2}ml^2 \dot{\theta}^2
$$

---

## 5. 势能 (Potential Energy)

设零势能在摆杆水平位置，势能为：

$$
V = - mgl \cos\theta
$$

---

## 6. 拉格朗日量

$$
L = T - V
$$

$$
L = \frac{1}{2}(M+m)\dot{x}^2 + ml\cos\theta \dot{x}\dot{\theta} + \frac{1}{2}ml^2 \dot{\theta}^2 + mgl \cos\theta
$$

---

## 7. 拉格朗日方程

对每个广义坐标（$x, \theta$）写方程：

### (1) 对 $x$：

$$
\frac{d}{dt}\left(\frac{\partial L}{\partial \dot{x}}\right) - \frac{\partial L}{\partial x} = Q_x
$$

计算：

$$
\frac{\partial L}{\partial \dot{x}} = (M+m)\dot{x} + ml\cos\theta \dot{\theta}
$$

$$
\frac{d}{dt}\left(\frac{\partial L}{\partial \dot{x}}\right) = (M+m)\ddot{x} + ml\cos\theta \ddot{\theta} - ml\sin\theta \dot{\theta}^2
$$

$\frac{\partial L}{\partial x} = 0$，所以：

$$
(M+m)\ddot{x} + ml\cos\theta \ddot{\theta} - ml\sin\theta \dot{\theta}^2 = u
$$

---

### (2) 对 $\theta$：

$$
\frac{d}{dt}\left(\frac{\partial L}{\partial \dot{\theta}}\right) - \frac{\partial L}{\partial \theta} = 0
$$

计算：

$$
\frac{\partial L}{\partial \dot{\theta}} = ml\cos\theta \dot{x} + ml^2 \dot{\theta}
$$

$$
\frac{d}{dt}\left(\frac{\partial L}{\partial \dot{\theta}}\right) = ml\cos\theta \ddot{x} - ml\sin\theta \dot{\theta}\dot{x} + ml^2 \ddot{\theta}
$$

$$
\frac{\partial L}{\partial \theta} = -ml\sin\theta \dot{x}\dot{\theta} - mgl\sin\theta
$$

所以：

$$
ml\cos\theta \ddot{x} + ml^2 \ddot{\theta} - mgl\sin\theta = 0
$$

---

## 8. 最终方程

得到系统动力学方程：

$$
(M+m)\ddot{x} + ml\cos(\theta)\ddot{\theta} - ml\sin(\theta)\dot{\theta}^2 = u
$$

$$
ml\cos(\theta)\ddot{x} + ml^2 \ddot{\theta} - mgl\sin(\theta) = 0
$$

这正是你图片里给出的 **倒立摆方程 (1)**。

---
## II.C Linearized Model
## 9. 状态空间形式

好——下面我把 **IPC（手推车倒立摆）** 的状态空间方程给出完整流程：先写 **非线性（精确）状态方程**，再给出在竖直向上平衡点处的 **线性化状态空间形式**（便于做 LQR / 线性控制设计）。

我们从之前得到的两条运动方程开始（与前面一致）：

$$
(M+m)\,\ddot x + m l\cos\theta\,\ddot\theta - m l\sin\theta\,\dot\theta^{2} = u,
\tag{1}
$$

$$
m l\cos\theta\,\ddot x + m l^{2}\,\ddot\theta - m g l\sin\theta = 0.
\tag{2}
$$

## 1) 写成矩阵形式并解出 $\ddot x,\ddot\theta$

把 (1),(2) 写成矩阵形式：

$$
\begin{bmatrix}
M+m & m l\cos\theta \\
m l\cos\theta & m l^2
\end{bmatrix}
\begin{bmatrix} 
\ddot x \\ \ddot\theta
\end{bmatrix} =
\begin{bmatrix}
u + m l\sin\theta \dot\theta^2 \\
m g l\sin\theta
\end{bmatrix}
$$

设矩阵的行列式（记为 $\Delta$）为

$$
\Delta = (M+m)\,m l^2 - (m l\cos\theta)^2
     = m l^2\big(M + m\sin^2\theta\big).
$$

用克拉默法则或直接求逆，得到显式解：

$$
\ddot x = \frac{1}{\Delta}\Big( m l^2\big(u + m l\sin\theta\,\dot\theta^2\big)
            - m l\cos\theta\,(m g l\sin\theta)\Big),
$$

$$
\ddot\theta = \frac{1}{\Delta}\Big( (M+m)(m g l\sin\theta)
            - m l\cos\theta\big(u + m l\sin\theta\,\dot\theta^2\big)\Big).
$$

你也可以把分子展成更直观的项（惯性耦合项、重力项、非线性项等），但上面形式已经清晰表明分母是质量矩阵的行列式，数值上始终非零（只要参数合理）。

---

## 2) 定义状态向量（非线性状态空间）

令状态向量

$$
\mathbf{x} = \begin{bmatrix} x_1\\ x_2\\ x_3\\ x_4\end{bmatrix}
= \begin{bmatrix} x\\ \dot x\\ \theta\\ \dot\theta\end{bmatrix},
\quad\text{输入 } u.
$$

则**非线性**状态方程为

$$
\dot x_1 = x_2,
$$

$$
\dot x_2 = \ddot x(\,x_3,x_4,u\,)\quad\text{按上面的显式表达},
$$

$$
\dot x_3 = x_4,
$$

$$
\dot x_4 = \ddot\theta(\,x_3,x_4,u\,)\quad\text{按上面的显式表达}.
$$

这就是完整的非线性状态空间模型，可直接用于数值仿真或非线性控制（非线性 MPC、反馈线性化等）。

---

## 3) 在竖直向上平衡点处（$\theta=0,\ \dot\theta=0$）做线性化

控制设计常取 **小角近似** $\sin\theta\approx\theta,\ \cos\theta\approx 1$，并忽略二阶小量 $\dot\theta^2$。先把质量矩阵在 $\theta\approx0$ 处简化为

$$
A_0=\begin{bmatrix}M+m & m l\\ m l & m l^2\end{bmatrix},\qquad
\Delta_0 = m l^2 M.
$$

线性化后解得（直接计算或用上面逆矩阵）：

$$
\ddot x \approx \frac{1}{M}\,u - \frac{m g}{M}\,\theta,
$$

$$
\ddot\theta \approx -\frac{1}{M l}\,u + \frac{(M+m)g}{M l}\,\theta.
$$

因此线性化的状态空间（以状态 $[x,\dot x,\theta,\dot\theta]^T$）有标准矩阵形式

$$
\dot{\mathbf{x}} = A\mathbf{x} + B u,
$$

其中

$$
A = \begin{bmatrix}
0 & 1 & 0 & 0\\[4pt]
0 & 0 & -\dfrac{m g}{M} & 0\\[6pt]
0 & 0 & 0 & 1\\[6pt]
0 & 0 & \dfrac{(M+m)g}{M l} & 0
\end{bmatrix},\qquad
B = \begin{bmatrix}
0\\[4pt]
\dfrac{1}{M}\\[6pt]
0\\[6pt]
-\dfrac{1}{M l}
\end{bmatrix}.
$$

（注意：上式中 $\theta$ 是相对于竖直向上的小偏角 —— 这是倒立摆常用线性化。）


