(hank_zlb)=

# 求解非线性转移路径：存在零利率下限的前瞻性指引（McKay，Nakamura & Steinsson，2016）

在这个应用中，我们求解一个单一资产的异质性主体新凯恩斯模型（McKay，Nakamura & Steinsson，2016）。这篇论文主要研究货币政策在存在零利率下限（即名义利率不能低于零）时的前瞻性指引作用。在文章中，对家庭贴现率的正向冲击会引发流动性陷阱；前瞻性指引是指在名义利率不再受到零下限约束时，中央银行许诺延长零利率的时限。

这个应用展现了工具包求解高度非线性模型的能力。当经济体处于流动性陷阱时，均衡经济系统与常规情况很不相同，这使得采用传统方法求解模型非常困难。

我们采用和原文相同的数学符号和参数设定。

## 模型

### 家庭问题

每个家庭成员在劳动生产率 $e$ 和债券持有 $b$ 上存在异质性。其中，劳动生产率的变化服从一个马尔科夫过程。家庭成员求解如下的贝尔曼方程：

$$
V_t(e,b) = \max_{c,n,b'} \frac{c^{1-\gamma}}{1-\gamma} - \chi \frac{n^{1+1/\nu}}{1+1/\nu}+ \beta_t \mathbb{E}[V_{t+1}(e',b')|e]
\\
s.t. \quad c+\frac{b'}{1+r_t}=b + w_ten - \tau_t \bar{\tau}(e) +D_t
\\
b' \geq 0,n\geq 0
$$

其中，$r_t$ 表示实际利率，$w_t$ 表示工资率，$\tau_t$ 表示边际税率，$D_t$ 表示来自企业的总利润。$\bar{\tau}(e)$ 的取值（0或1）表示拥有生产率 $e$ 的劳动者是否被征税。家庭成员选择最优的消费 $c$，劳动供给 $n$，债券持有 $b'$，并且受到借贷约束 $b' \geq 0$。

个体状态变量的分布由 $\Gamma_t$ 表示。个体的决策规则由 $g_{c,t}, g_{n,t}, g_{b',t}$ 表示。

### 新凯恩斯设定
部分垄断竞争的厂商雇佣劳动力生产异质性中间投入品，这些中间投入品会被竞争性厂商用于生产最终消费品。最终消费品的生产函数是 CES。中间投入品的价格存在粘性，即每期只有一部分的中间品生产商可以调整价格。这种标准的新凯恩斯模型设定可以推导出一个标准的菲利普斯曲线：
$$
Y_tS_t = z_t  N_t
\\
S_{t} =\theta (\frac{p_{t}^*}{P_{t}})^{\mu / (1-\mu)}+(\frac{1}{1+\pi_{t}})^{\mu / (1-\mu)} (1-\theta)S_{t-1}
\\
1+\pi_{t} =( \frac{1-\theta}{1- \theta( \frac{p_{t}^*}{P_{t}})^{\frac{1}{1-\mu}}})^{1-\mu}
\\
\frac{p_t^*}{P_t} = \frac{P_t^A}{P_t^B}
\\
P_t^A =\mu\frac{w_{t}}{z_{t}}Y_{t} + (1-\theta)\beta (1+\pi_{t+1})^{-\mu/(1-\mu)}P_{t+1}^A
\\
P_t^B =Y_{t} + (1-\theta)\beta (1+\pi_{t+1})^{-1/(1-\mu)}P_{t+1}
$$

其中，$\mu$ 表示边际替代弹性，$\theta$ 表示每一期中间产品生产商能够调整价格的概率。$Y_t$ 是总产出。$S_t$ 是价格的离散程度。$N_t$ 是劳动力总需求。$z_t$ 是外生的劳动生产率。$\pi_t$ 是通货膨胀率。$p_t^*$ 是能够调整价格的厂商的最优价格。$P_t^A，P_t^B$ 是用于求解模型的辅助变量。

中央银行依照泰勒规则设置名义利率，并且受到名义利率不能低于零的约束：

$$
i_t = \max\{i_{0} + \phi \pi_t,0\}
$$

当不存在宏观不确定性时，费雪方程成立：

$$
1+r_t=\frac{1+i_t}{1+\pi_{t+1}}
$$

政府部门通过调整税率 $\tau_t$ 来满足外生的政府支出和债券的利息支出。

$$
\tau_t =\frac{B_t + G_t - \frac{B_{t+1}}{1+r_{t}}}{\int \bar{\tau}(e) \ \mathrm{d}\Gamma_t}
$$

其中，$B_t$ 表示期初的政府债券规模；$G_t$ 表示政府开支总额。

在基准模型中，财政支出的规则是政府债券总额不变：$B_{t+1}=\overline{B}$。

### 市场出清条件

债券市场出清条件:

$$
B_{t+1}=\int g_{b',t}(e,b) \ \mathrm{d}\Gamma_t
$$

劳动力市场出清条件:

$$
N_t = \int eg_{n,t}(e,b) \ \mathrm{d}\Gamma_t
$$

商品市场出清条件:

$$
\int g_{c,t}(e,b) \ \mathrm{d} \Gamma_t + G_t = Y_t
$$


### 均衡的定义

给定家庭状态的初始分布 $\Gamma_0$，序列竞争均衡由以下变量刻画：（1）家庭个体状态变量的分布函数：$\{\Gamma_t\}_{t=0}^{\infty}$；（2）家庭的值函数与策略函数 $\{V_t,g_{c,t},g_{n,t},g_{b',t}\}_{t=0}^{\infty}$； (3) 总体数量和价格 $\{w_t,Y_t,S_t,N_t,D_t,\pi_t,\frac{p_t^*}{P_t},P_t^A,P_t^B,r_t,i_t,\tau_t\}_{t=0}^{\infty}$ 。这些序列满足如下条件：

$$
Y_t S_t= z_t N_t
\\
D_t=Y_t-w_tN_t
\\
S_{t} =\theta (\frac{p_{t}^*}{P_{t}})^{\mu / (1-\mu)}+(\frac{1}{1+\pi_{t}})^{\mu / (1-\mu)} (1-\theta)S_{t-1}
\\
1+\pi_{t} =( \frac{1-\theta}{1- \theta( \frac{p_{t}^*}{P_{t}})^{\frac{1}{1-\mu}}})^{1-\mu}
\\
\frac{p_t^*}{P_t} = \frac{P_t^A}{P_t^B}
\\
P_t^A =\mu\frac{w_{t}}{z_{t}}Y_{t} + (1-\theta)\beta (1+\pi_{t+1})^{-\mu/(1-\mu)}P_{t+1}^A
\\
P_t^B =Y_{t} + (1-\theta)\beta (1+\pi_{t+1})^{-1/(1-\mu)}P_{t+1}^B
\\
1+r_t=\frac{1+i_t}{1+\pi_{t+1}}
\\
i_t = \max\{i_{0} + \phi \pi_t,0\}
\\
\tau_t =\frac{B_t + G_t - \frac{B_{t+1}}{1+r_{t}}}{\int 1(e=e_{High}) \Gamma_t}
\\
N_t = \int eg_{n,t}(e,b) d\Gamma_t
\\
\overline{B}=\int g_{b',t}(e,b) d\Gamma_t
$$

商品市场出清条件满足瓦尔拉斯定理。这个系统还可以进一步简化，参见 hmod 文件。

## hmod 文件

模型可以由 hmod 文件表示，{download}`hank1.hmod`，如下所示：

```{literalinclude} hank1.hmod
:language: HANS
:linenos:
```

均衡系统现在包含了非平凡的市场出清条件，但是仍然可以被表示成方程组。在 **var_agg** 中定义所有未知数的时间序列，并且在 **model** 模块中用 "==" 定义相同数量的方程 (见下面代码中的高亮部分):

```{literalinclude} hank1.hmod
:language: HANS
:linenos:
:lineno-match:
:start-at: var_agg Y
:end-at: end;
:emphasize-lines: 1, 28-33
```

可以定义一个不同于主 **model** 模块的替代模型块。例如，脚本文件中的以下模块定义了用于校准的块，它采用不同的未知数和不同的方程组。校准在这里处于稳态，因此我们可以将校准系统简化为总劳动力和债券持有的方程，所有其他变量都是两个未知数的简单函数，或者由稳态条件隐含。 

```{literalinclude} hank1.hmod
:language: HANS
:linenos:
:lineno-match:
:start-at: model_cali(
:end-at: end;
:emphasize-lines: 1, 8-9
```

所有的 **var_agg** 都需要在声明之后进行正确的初始化，除非它们的值是由替代模块返回的 (即, 此处的 **model_cali** 模块)。关于如何通过传递解出的校准来求解主模型块，请参见下文。

## 使用工具箱

在解析完脚本文件之后，工具箱生成了 MATLAB 文件：*solve_vfi.m*，*solve_ss.m*，*solve_trans_linear.m*，*solve_trans_nonlinear.m*，*solve_cali.m* 以及其他可用于求解稳态、平稳分布、转移路径和替代系统 (在此处是用于校准) 的函数。生成文件的使用如下所示。

```{raw} html
:file: hank1_zlb_notebook.html
```

```{raw} latex
\includepdf[pages=-]{../../source/examples/HANK1_ZLB/hank1_zlb_notebook.pdf}
```
