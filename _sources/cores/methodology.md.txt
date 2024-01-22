(toolbox_algorithm)=
# 算法

## 通用框架

本文构建了一个通用框架，通过总体内生变量方程组表示各类异质性主体模型的序列均衡。$\mathbb{Z} \subseteq \mathbb{R}^{N_Z}$，$\mathbb{Y} \subseteq \mathbb{R}^{N_Y}$，$\mathbb{E}\subseteq  \mathbb{R}^{N_e}$，$\mathbb{A}\subseteq  \mathbb{R}^{N_a}$，$\mathbb{X} \subseteq \mathbb{R}^{N_x}$ 分别表示外生总体变量、内生总体变量、外生个体状态变量、内生个体状态变量以及个体策略变量。 

取一个较大的整数截断时期 $T$，以 $\Phi_0$ 作为初始的个体状态分布 $\mathbb{E}\times \mathbb{A}$ 以及外生总体变量序列 $\mathcal{Z}\equiv \{Z_t\}_{t=0}^{T} \in \mathbb{Z}^{T+1}$ 的度量，一个均衡可以由一组内生总体变量 $\mathcal{Y}\equiv\{Y_t\}_{t=0}^{T} \in \mathbb{Y}^{T+1}$ 的方程来表示:

$$
\mathcal{F}_t\left(\{Z_s\}_{s=0}^{t},\{Y_s\}_{s=0}^{t},\bar{x}_t(\mathcal{Z},\mathcal{Y})\right)=0, \forall t=0,1,...,T
$$

```{math}
:label: F_general
```

其中，$\mathcal{F}_t：\mathbb{Z}^{T+1} \times \mathbb{Y}^{T+1} \times \mathbb{X} \rightarrow \mathbb{R}^{N_Y}$ 是一个从外生总体变量、内生总体变量以及个体策略变量到均衡条件残差的映射。

$\bar{x}_t(\mathcal{Z},\mathcal{Y})$ 是时期 $t$ 个体决策变量的加总，这个加总是通过对个体状态分布进行积分得到的。具体来说，

$$
\bar{x}_t(\mathcal{Z},\mathcal{Y}) = \int g_{x,t}(e,a;\mathcal{Z},\mathcal{Y}) \ \mathrm{d}\Phi_t(e,a;\mathcal{Z},\mathcal{Y}),
$$

其中，$g_{x,t}(e,a;\mathcal{Z},\mathcal{Y})$ 是求解贝尔曼方程的策略函数：

$$
v_t(e,a;\mathcal{Z},\mathcal{Y}) = \max_{x} u(e,a,x;Z_t,Y_t) + \beta \mathbb{E}[v_{t+1}(e',a';\mathcal{Z},\mathcal{Y})|e,a,x]
\\
s.t. \quad \mathcal{C}(e,a,x; Z_t,Y_t) \leq 0
\\
a' \sim \mathcal{T}(\cdot;e,a,x; Z_t,Y_t)
\\
v_{T+1} \text{ 由稳态值函数决定}
$$

其中，$u(e,a,x;Z_t,Y_t)$ 是当期效用函数。 $\mathcal{C}(e,a,x; Z_t,Y_t)\leq 0$ 是最优化问题面临的一组不等式约束。$\mathcal{T}$ 是一个关于 $x'$ 的分布，表示个体内生状态变量的（可能是随机的）转移。

$\Phi_t(e,a;\mathcal{Z},\mathcal{Y})$ 是一个度量，代表时期 $t$ 个体状态的分布，这个分布根据策略函数和马尔科夫过程 $e$ 的外生转移进行演变，外生转移测度由 $P^e$ 表示：

$$
\Phi_{t+1}(\mathcal{E}',\mathcal{A}';\mathcal{Z},\mathcal{Y})= \int  P^e(\mathcal{E}'|e)\mathcal{T}\Big(\mathcal{A}';e,a,g_{x,t}\Big(e,a;\mathcal{Z},\mathcal{Y}\Big);Z_t,Y_t\Big) \Phi_t (d e,d a;\mathcal{Z},\mathcal{Y})
$$

### 例子：Krusell & Smith (1998)

将 Krusell & Smith（1998）模型应用于通用框架，左侧变量表示框架符号，右侧变量是模型变量，即 $Z_t=Z_t$，$Y_t=K_{t+1}$，$e=e$，$a=a$，$x=a'$。

$$
\mathcal{F}_t\left(\{Z_s\}_{s=0}^{T},\{K_s\}_{s=0}^{T},\bar{x}_t(\mathcal{Z},\mathcal{Y})\right)=K_{t+1}-\bar a'_{t}
$$

请注意，$K_0=\int a d \Phi_0(de,da)$ 为已知的，因此不属于未知变量。

当期效用函数:

$$
u(e,a,a';Z_t,K_t)=\ln\Big([1+r(Z_t,K_t)]a + w(Z_t,K_t)[(1.0-\tau)e\bar{l} + \mu (1- e)]-a'\Big)
$$

两个不等式约束:

$$
\mathcal{C}^{(1)}(e,a,a'; Z_t,K_t) = -a'
\\
\mathcal{C}^{(2)}(e,a,a'; Z_t,K_t) = a' -[1+r(Z_t,K_t)]a + w(Z_t,K_t)[(1.0-\tau)e\bar{l} + \mu (1- e)]
$$

状态转移:

$$
\mathcal{T}(\cdot;e,a,a'; Z_t,K_t)=a' \text{ 依概率1}
$$

## 使用局部 Broyden 更新计算雅各比矩阵（Jacobian） 

尽管方程系统 {eq}`F_general` 规模庞大，但只要能够有效计算关于内生总体变量 ${Y}_{t=0}^{T}$ 的雅各比矩阵，就可以利用基于梯度的方程求解器高效地解决问题。计算 Jacobian 矩阵可能具有挑战性，因为求解该系统需要先根据贝尔曼方程求解策略函数，再求解个体状态分布。有限差分方法效率较低，因为在每次沿着各个维度扰动总体变量时都需要重新求解策略函数及相应的状态转移。自动微分方法（AD）也适用于这种情况，因为总体变量在个体最优化问题中是作为参数来处理的。

序列空间雅各比（Sequence Space Jacobian）算法（[Auclert 等，2021](https://onlinelibrary.wiley.com/doi/full/10.3982/ECTA17434)）提出了一种计算模型稳态雅各比矩阵的有效途径，并建议在求解转移路径时固定雅各比矩阵。这种方法能够充分描述模型在稳态附近的动态，但不适用于具有高度非线性特征的模型，例如具有零利率下限的新凯恩斯模型（McKay，Nakamura & Steinsson，2016）。

本算法的核心贡献在于提供了雅各比矩阵的稀疏更新，从而增强了算法的稳健性与有效性。定义 $\mathcal{G}_t(\mathcal{Z},\mathcal{Y})=\mathcal{F}_t\left(\{Z_s\}_{s=0}^{t},\{Y_s\}_{s=0}^{t},\bar{x}_t(\mathcal{Z},\mathcal{Y})\right)$，$\{Y_s\}$ 的雅各比矩阵元素可以表示为：

$$
\nabla_{Y_s} \mathcal{G}_t=\nabla_{Y_s} \mathcal{F}_t + \nabla_{\bar{x}_t} \mathcal{F}_t \cdot  \nabla_{Y_s} \bar{x}_t
$$

关键的发现是 $\nabla_{Y_s} \mathcal{F}_t$ 和 $\nabla_{\bar{x}_t} \mathcal{F}_t$ 可以通过解析方法或者自动微分方法计算得出。因此，在知道 $\nabla_{Y_s} \bar{x}_t$ 的稳态值之后，我们只需要在求解过程中对 $\nabla_{Y_s} \bar{x}_t$ 进行更新。本算法汲取了 Schubert （1970）中的思想，对 $\nabla_{Y_s} \bar{x}_t$ 进行稀疏 Broyden 更新。这种稀疏更新方法依赖于用户输入来识别雅各比矩阵中的 $0$ 元素，并且只对剩余的非 $0$ 元素进行更新。与标准的 Broyden 方法类似，在搜索步骤中，除函数值之外不再需要其他信息。稀疏结构由工具包根据模型结构自动构建。稀疏更新既是必要的又是有效的，因为并非所有总体变量都会影响个体决策。而传统的、不考虑稀疏性的稠密 Broyden 更新可能会导致错误的稀疏结构，在很多情况下，会导致方程求解发散或性能不佳。

初始的稳态雅各比矩阵 $\nabla_{Y_s} \bar{x}_t$ 是通过虚拟消息算法（Fake News Algorithm）构建的（[Auclert 等，2021](https://onlinelibrary.wiley.com/doi/full/10.3982/ECTA17434)）。

## 其他实现细节

为了确保最佳性能，求解个体最优化问题的程序被编译成 C++ 库（MATLAB 中的 mex 二进制文件）。函数通过线性插值或者三次样条插值来近似，并且可以选择指定外插的不同阶数。最优化问题由高效的适用于小型问题的数值算法求解。在优化问题中尽可能的使用了解析解梯度，并以自动微分作为备选方案。

根据 Young（2010），个体状态分布及其确定性转移在直方图上近似。平稳分布是通过求解个体状态的平稳转移矩阵的特征向量得到的。在构建过程中考虑了转移矩阵的稀疏性，以确保性能。

总体均衡系统在 MATLAB 中定义。为稳态和转移路径提供了多种方程求解算法，并按照上述步骤构造雅各比矩阵。

工具箱还提供了个体样本的随机模拟功能。

关于最优化及方程求解算法的列表，请参阅 [工具箱 API 参考](toolbox_api)。





