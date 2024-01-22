(ks_1998)=
# 入门：Krusell & Smith（1998）

该模型描述了一个生产经济，其中家庭面临无法保险的个体异质性劳动收入风险。总体冲击是对全要素生产率的冲击。工具包能够求解模型稳态，并且可以求解未预期的总体冲击发生后确定性的线性/非线性转移路径。根据 [Boppart，Krusell & Mitman (2018)](https://www.sciencedirect.com/science/article/abs/pii/S0165188918300022) 和 [Auclert 等，（2021）](https://onlinelibrary.wiley.com/doi/full/10.3982/ECTA17434)，确定性转移路径刻画了总体冲击的一阶效应。

为便于比较，我们采用了 [Den Haan，Judd & Juillard (2010)](https://www.sciencedirect.com/science/article/pii/S0165188909001286) 中的模型设定及参数值。

## 模型

家庭具有异质性的就业状态 $e\in\{0,1\}$ 及资本持有 $a \in R^+$，并求解如下的贝尔曼方程：

$$
v_t(e,a) = \max_{c,a'} \log(c)+ \beta\mathbb{E}[v_{t+1}(e',a')|e]
\\s.t. \quad c+a' \leq  (1+r_t)a +  [(1-\tau)\overline{l}e + \mu(1-e)]w_t
\\a'\geq 0, c \geq 0,
$$
```{math}
:label: bellman
```

其中 $\bar{l}$ 是就业状态下的劳动供给，$\tau$ 是劳动所得税率，$\mu$ 是失业保险偿付率。$r_t$ 和 $w_t$ 分别为利率和工资。家庭面临无借贷约束 $a'\geq 0$。

代表性厂商使用资本与劳动进行生产：$Y_t = Z_t K_t^{\alpha} L_t^{1-\alpha}$。资本折旧率为 $\delta$。利率和工资由市场竞争决定。

$$
r_t = \alpha Z_t K_t^{\alpha-1}L_t^{1-\alpha} - \delta
\\
w_t = (1-\alpha)Z_tK_t^{\alpha}L_t^{-\alpha}
$$
```{math}
:label: competitive_prices
```

给定家庭状态的初始分布 $\Phi_0$，序列竞争均衡定义为：（1）家庭状态分布 $\{\Phi_t\}_{t=0}^{\infty}$；（2）家庭值函数与策略函数 $\{v_t,g_{c,t},g_{a',t}\}_{t=0}^{\infty}$；（3）总体数量与价格 $\{K_t, L_t, r_t, w_t\}_{t=0}^{\infty}$ 的序列，使得：

- $\{v_t,g_{c,t},g_{a',t}\}$ 求解由 {eq}`bellman` 定义的家庭最优化问题；
- 市场出清: $K_t=\int a \ \mathrm{d}\Phi_t(e,a)$, $L_t=\int \bar{l} e \ \mathrm{d}\Phi_t(e,a)$。商品市场出清满足瓦尔拉斯定律。
- $r_t$ 和 $w_t$ 由 {eq}`competitive_prices` 确定。
- 失业救济金来源于劳动所得税：$w_t\int \bar{l} \tau e \ \mathrm{d}\Phi_t(e,a)=w_t\int (1-e)\mu \ \mathrm{d}\Phi_t(e,a)$。
- $\{\Phi_t\}$ 与策略函数 $g_{a',t}$ 和 $e$ 的外生转移一致。

## 工具箱脚本（hmod）文件

模型可以由 {download}`KS_JEDC10.hmod` 描述，如下所示：

```{literalinclude} KS_JEDC10.hmod
:language: HANS
:linenos:
```

脚本文件的目标是定义一个 **vfi**（值函数迭代）模块，用于确定个体决策问题，并定义一个 **model** 模块，用于确定总体均衡条件。

在 **vfi** 模块之前需要定义用于求解个体问题所需的信息。模块中定义的信息将按照顺序进行解释。

```{literalinclude} KS_JEDC10.hmod
:language: HANS
:linenos:
:lineno-match:
:start-at: parameters
:end-before: var_shock
```

该模块声明了 **vfi** 模块中使用的参数及其数值。

```{literalinclude} KS_JEDC10.hmod
:language: HANS
:linenos:
:lineno-match:
:start-at: var_shock
:end-before: var_state
```

该模块在 *var_shock* 中声明了个体外生状态及其数值，这些状态需要被定义为具有联合转移矩阵 *shock_trans* 的离散马尔科夫过程。给定失业保险偿付率 $\mu$ 的校准，平衡预算的劳动所得税率 $\tau$ 可以预先确定。

```{literalinclude} KS_JEDC10.hmod
:language: HANS
:linenos:
:lineno-match:
:start-at: var_state
:end-before: var_pre_vfi
```

该模块在 *var_state* 中声明了个体的内生状态，并且指定了用于近似值函数及策略函数的离散格点值。对于多维状态，函数将在离散状态值的张量积上进行近似。

```{literalinclude} KS_JEDC10.hmod
:language: HANS
:linenos:
:lineno-match:
:start-at: var_pre_vfi
:end-before: var_policy
```

该模块定义了一些简单变量，这些变量是外生状态和/或内生状态的函数，可以在求解决策问题之前确定。在这里，预算被视为按照特定价格（价格被定义为参数）计算的资本与劳动收入之和。该模块是可选的。

```{literalinclude} KS_JEDC10.hmod
:language: HANS
:linenos:
:lineno-match:
:start-at: var_policy
:end-before: var_aux
```

将需要优化的变量（此处为未来资本持有 $a'$）在 *var_policy* 中声明。辅助变量（此处为消费 $c$）可以直接作为 *var_shock*，*var_state* 和/或 *var_policy* 的简单函数进行计算，并被声明为 *var_aux*。

所有信息准备就绪之后，在 **vfi** 模块中的 *vfi;* 与 *end;* 之间定义个体问题。

```{literalinclude} KS_JEDC10.hmod
:language: HANS
:linenos:
:lineno-match:
:start-at: vfi;
:end-at: end;
:emphasize-lines: 4
```
**vfi** 模块从 *var_state*（此处为 $(e,a)$）和 *var_policy*（此处为 $a'$）的值开始，定义 *Tv* 中的目标（被标注的行）。如所示，这一行遵循贝尔曼方程的自然语法，其中未来值函数 *v* 被视为已知，并在未来的内生状态值上调用。*EXPECT* 操作符用于求解未来外生状态的积分。

*var_policy* 的约束（此处为 $0 \leq a'\leq budget$）以及任意的等式或不等式约束可以在定义目标之后进行说明。 

所有的 *var_aux*（此处为 $c$）都需要被定义。

脚本文件可以在定义 **vfi** 块后停止，因此工具箱只生成用于解决自包含决策问题的代码。 

为了求解均衡，需要在 **model** 模块中定义一个总体变量方程组，在模块前定义的全部所需信息将按照顺序解释。

```{literalinclude} KS_JEDC10.hmod
:language: HANS
:linenos:
:lineno-match:
:start-at: var_agg
:end-before: var_agg_shock
```

均衡系统的未知数需要声明为 *var_agg*（此处为 $K$），并指定其初始值。定义总体系统时使用的任何参数值（此处为 $\alpha，\delta$）也需要被明确指定。

```{literalinclude} KS_JEDC10.hmod
:language: HANS
:linenos:
:lineno-match:
:start-at: var_agg_shock
:end-before: model;
```

总体冲击被声明为 *var_agg_shock*。总体冲击本质上是模型参数，并允许随时间变化。此处为总体 TFP，$Z$。

所有信息准备就绪之后，总体均衡系统被定义为：

```{literalinclude} KS_JEDC10.hmod
:language: HANS
:linenos:
:lineno-match:
:start-at: model;
:end-at: end;
:emphasize-lines: 5
```

**model** 模块的目标是定义表征总体均衡条件的方程系统。每个方程都用 "==" 定义，如代码块中突出显示的那样。在这里，系统由一个简单的方程组成，即资产市场出清条件 $K_{t+1}=\int g_{a',t} d\Phi_t(e,a)$。*var_policy* 可用于方程定义之中，表示在 $t$ 时刻基于分布的个体策略函数加总，行中的 *ap* 表示 $\int g_{a',t} d\Phi_t(e,a)$（$ap$ 被声明为 *var_policy*）。

在定义系统之前，需要按顺序计算中间变量 $r，w$。请注意，在定义方程之前，需要更新用于解决个体决策问题的参数（此处为 $r$，$w$），以适应 *var_agg* 中的变化。对于变量的时间序列索引，符号遵循 Dynare 的惯例：将预先确定的总体变量作为滞后变量进行索引（此处使用 $K(-1)$ 计算 $r$ 和 $w$）。

可以对 *var_agg* 施加约束。这里指定了 $K \geq 38$ 使得 $r$ 有界，以确保值函数迭代能够收敛。

定义方程后，可以定义其他的总体均衡变量（此处最后三行定义了 $Y，I，C$）。求解系统后会返回这些变量值。

## 使用工具箱

解析脚本文件后，工具箱会生成 MATLAB 文件，包括：*solve_vfi.m*，*solve_ss.m*，*solve_trans_linear.m*，*solve_trans_nonlinear.m* 以及其他可以调用的函数，用于求解模型的稳态、平稳分布和转移路径。这些函数的使用方法如下所示。

```{raw} html
:file: ks_notebook.html
```

```{raw} latex
\includepdf[pages=-]{../../source/examples/KS_JEDC10/ks_notebook.pdf}
```

## 接下来做什么？

了解[工具箱算法](toolbox_algorithm)的详细信息。

查看[工具箱 API 参考](toolbox_api)。

更多示例：

-  [McKay, Nakamura 和 Steinsson (2016)](hank_zlb): 包含异质性主体的新凯恩斯模型（HANK），用于定义复杂的均衡条件和处理非线性问题
- 一个用于处理投资组合选择的[双资产 HANK 模型](hank2_ssj)
-  [Khan 和 Thomas (2008)](kt2008) : 处理非凸调整成本和随机状态转移
