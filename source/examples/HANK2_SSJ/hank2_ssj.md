(hank2_ssj)=
# 求解资产组合：两资产 HANK 模型

在这个例子中，我们求解了 Auclert, Bardóczy, Rognlie, and Straub (2021, 附录 B.3 和 附录 E.1) 中的两资产 HANK 模型。模型中，每个家庭选择最优的流动性与非流动性的资产组合，并面临着一个凸的资产组合调整成本。此外，新凯恩斯粘性价格企业决定中间产品的价格，面临着 Rotemberg-类型的价格调整成本，并且根据二次型的资本调整成本作出动态的投资决策。工会通过最大化其成员的平均福利来控制工资水平。

这个例子展示了该工具箱解决具有多维内生状态变量、控制变量以及大型均衡方程系统的复杂投资组合问题的能力。

我们遵循原文中的符号、参数化和校准程序。关于均衡方程组的详细推导，建议读者阅读原文。

## 模型

### 家庭

家庭具有异质性的马尔可夫劳动效率 ($e$)、初始流动资产持有量 ($b$) 和非流动性资产持有量 ($a$)，并求解贝尔曼方程：

$$
V_t\left(e_{it}, b_{it-1}, a_{it-1}\right) =\max_{c_{it}, b_{it}, a_{it}}\left\{\frac{c_{it}^{1-\sigma}}{1-\sigma}  - \varphi \frac{N_{t}^{1+\nu}}{1+\nu} +\beta \mathbb{E}_t V_{t+1}\left(e_{it+1}, b_{it}, a_{it}\right)\right\}
\\
\quad s.t. \quad c_{it} + a_{it} + b_{it} = \left(1-\tau_{t}\right)w_t e_{it} N_{t} + \left(1+r_{t}^{a}\right) a_{it-1}
\\
 +\left(1+r_t^b\right) b_{it-1}-\Phi_{t}\left(a_{it}, a_{it-1}\right)
\\
a_{it} \geq 0, \quad b_{it} \geq \underline{b}
$$

其中，$\tau_t$ 表示统一税率，$w_t$ 为真实工资，$N_t$ 为劳动时间，$r_t^b$ 和 $r_t^{a}$ 分别为流动性资产和非流动性资产的真实利率，$\Phi_t$ 为调整成本函数，定义为：

$$
\Phi_{t}\left(a_{it}, a_{it-1}\right)=\frac{\chi_1}{\chi_2}\left|\frac{a_{it}-\left(1+r_t^a\right) a_{it-1}}{\left(1+r_t^a\right) a_{it-1}+\chi_0}\right|^{\chi_2}\left[\left(1+r_t^a\right) a_{it-1}+\chi_0\right]
$$

其中 $\chi_0 > 0$，$\chi_1 > 0$，并且 $\chi_2 > 1$。

家庭选择消费 ($c_{it}$) 以及资产组合 $\left(b_{it}, a_{it}\right)$，使其满足借贷约束 $a_{it} \geq 0$ 和 $b_{it} \geq \underline{b}$，并提供一致的且由工会决定的劳动 ($N_{t}$) 以满足企业的劳动需求。

以 $\Gamma_t$ 表示家庭状态分布的度量，以 $g_{c,t}, g_{b,t}, g_{a,t}$ 表示策略函数。

### 金融中介

家庭将流动性资产和非流动性资产存入代表性金融中介机构。金融中介机构以单位流动性转换成本 $\omega$ 将流动性的存款 ($\int{b_{it}di}$) 投资于非流动性的政府债券之中，从而实现 $r_t^b = r_t - \omega$，其中 $r_t$ 表示政府债券在时期 $t$ 的实际利率。同时，金融中介机构还将非流动性存款 ($\int{a_{it}di}$) 投资于由政府债券和公司股权组成的共同基金之中，这意味着：

$$
1 + r_t^a = \Theta_{pt-1} \frac{D_t + p_t}{p_{t-1}} + (1 - \Theta_{pt-1}) \left(1 + r_t\right)
$$

其中，共同基金股权份额 $\Theta_{pt-1} = \frac{p_{t-1}}{p_{t-1} + B^{g}_{t-1} - B^{h}_{t-1}}$ 由时期 $t-1$ 的股权价格 ($p_{t-1}$)，总体政府债券 $\left(B^{g}_{t-1}\right)$, 以及总体流动性资产持有量 $\left(B^{h}_{t-1} = \int{b_{it-1}di}\right)$  决定。在完美预期均衡中，无套利条件意味着时期 $t \geq 1$ 未来回报的均等化。

$$
1 + r_t^a = \frac{D_t + p_t}{p_{t-1}} = 1 + r_t, \forall t \geq 1
$$

然而，$t = 0$ 的资产回报可能会因 $t = 0$ 发生的意外冲击而发生改变。

### 企业

中间品厂商使用柯布-道格拉斯生产技术，利用资本和劳动力生产差异化的产品。这些中间产品由竞争性的最终品厂商使用具有替代弹性 $\mu_{p}/\left(\mu_p - 1\right)$ 的 CES 聚合器进行加总。中间品厂商在中间产品市场上实行垄断竞争，使得动态投资决策服从于一个二次型的调整成本，并面临对数二次型的价格调整成本。

中间品厂商 $j$ 的需求、技术和调整成本可通过以下公式定义：

 - 产品需求函数：

$$
 y_{jt} = \left(\frac{p_{jt}}{P_{t}}\right)^{-\frac{\mu_{p}}{\mu_p - 1}} Y_{t}
$$

 - 生产函数：

$$
 y_{jt} = z_t k_{jt-1}^\alpha n_{jt}^{1-\alpha}
$$

  - 资本调整成本函数：

$$
 \zeta \left(k_{jt},k_{jt-1}\right) k_{jt-1} = \frac{1}{2 \delta \epsilon_I}\left(\frac{k_{jt}}{k_{jt-1}} - 1\right)^2 k_{jt-1}
$$

  - 价格调整成本函数：

$$
 \psi^p_t \left(p_{jt},p_{jt-1}\right)=\frac{\mu_p}{\mu_p-1} \frac{1}{2 \kappa_p}\left[\log \left(\frac{p_{jt}}{p_{jt-1}}\right)\right]^2 Y_t
$$

  - 资本积累方程：

$$
 i_{jt} = k_{jt} - \left(1 - \delta\right) k_{jt-1} + \zeta \left(k_{jt},k_{jt-1}\right) k_{jt-1}
$$

  - 股利方程：

$$
 d_{jt} = y_{jt} - w_{t} n_{jt} - i_{jt} - \psi^p_t \left(p_{jt},p_{jt-1}\right)
$$

在对称均衡中，企业的最优解可由以下方程表示：

 - 价格菲利普斯曲线：

$$
\log(1+\pi_t) = \kappa_p \left(mc_t - \frac{1}{\mu_p} \right) + \frac{1}{1+r_{t+1}} \frac{Y_{t+1}}{Y_t} \log(1+\pi_{t+1})
$$

 - 投资方程：

$$
Q_t = 1 + \frac{1}{\delta \epsilon_I}\left(\frac{K_t-K_{t-1}}{K_{t-1}}\right)
$$

- 估值方程：

$$
(1+r_{t+1})Q_{t} = \alpha \frac{Y_{t+1}}{K_t} mc_{t+1} - \left[\frac{K_{t+1}}{K_t} - (1-\delta) + \frac{1}{2\delta \epsilon_I}\left(\frac{K_{t+1} - K_t}{K_t}\right)^2\right] + \frac{K_{t+1}}{K_t}Q_{t+1}
$$
其中 $mc_t$ 是如下函数的简写： 

 - 边际成本函数：

$$
 mc_t = \frac{w_t }{(1-\alpha)\frac{Y_t}{N_t}} 
$$

### 工会

竞争性雇佣者加总了由垄断竞争工会连续统提供的差异化劳动力。每个工会 $k$ 面临一个具有替代弹性 $\mu_{w}/\left(\mu_{w} - 1\right)$ 的劳动需求函数，并设定名义工资率 ($W_{kt}$) 以最大化其成员的平均效用，并服从效用中的二次型调整成本。

$$
 \psi_t^w \left(W_{kt},W_{kt-1}\right)=\frac{\mu_w}{\mu_w - 1} \frac{1}{2 \kappa_w}\left[\log \left(\frac{W_{kt}}{W_{kt-1}}\right)\right]^2
$$

在对称均衡中，工会的最优解会得出工资菲利普斯曲线：

$$
\log \left(1+\pi_t^w\right)=\kappa_w\left[\varphi N_t^{1+v}-\frac{\left(1-\tau_t\right) w_t N_t}{\mu_w} \int{e_{it}c_{it}^{-\sigma}di}\right]+\beta \log \left(1+\pi_{t+1}^w\right)
$$
其中 $1 + \pi_t^w = W_t /W_{t-1} = (1 + \pi_t) w_t/w_{t-1}$ 是时期 $t$ 的工资通胀率。

### 货币政策与财政政策

货币当局实施泰勒规则，将稳定状态目标通胀率设定为零:

$$
i_t = r_{\ast} + \phi \pi_t + \phi_y \left(Y_t - Y_{\ast}\right) + m_t
$$

其中 $m_t$ 是货币政策冲击，$r_{\ast}$ and $Y_{\ast}$ 分别为稳态的真实利率与 GDP。

名义利率、真实利率与通胀率通过费雪方程相互关联：

$$
1+r_t=\frac{1+i_{t-1}}{1+\pi_{t}}
$$

政府通过实施预算平衡来维持不变的债务 ($B^g$) 实际价值，并通过税收 $\tau_t$ 来筹集外部购买支出 ($G_t$) 和债券利息支付所需的资金。

$$
\tau_t =\frac{r_{t}B^{g} + G_t}{w_t N_t}
$$

### 市场出清条件

 - 资本市场出清：

$$
p_{t} + B^{g}=\int{\left(a_{it} + b_{it}\right)di}
$$

- 商品市场出清：

$$
Y_t = \int{c_{it}di} + \int{\Phi_{t}\left(a_{it}, a_{it-1}\right)di} + \omega \int{b_{it}di}  + I_t + \psi^p \left(P_{t},P_{t-1}\right) + G_t
$$

注意，通过使用 $N_t$ 来表示企业问题中的劳动力需求和工资菲利普斯曲线中的劳动力供给，我们隐含地强加了劳动力市场出清条件。

### 均衡方程组：转移路径 

给定家庭状态的初始分布 $\Gamma_0$，完美预期均衡是：(1) 家庭状态分布 $\{\Gamma_t\}_{t=0}^{\infty}$；(2) 家庭值函数与策略函数 $\{V_t,g_{c,t},g_{b,t},g_{a,t}\}_{t=0}^{\infty}$；以及 (3) 总体数量与价格 $\{Y_t,K_t,N_t,I_t,\psi^{p}_{t},D_t,B_t^h,w_t,mc_t,p_t,Q_t,\pi_t,\pi_{t}^{w},i_t,r_t,r_t^a,r_t^b,\tau_t\}_{t=0}^{\infty}$ 的序列，使其满足：

$$
Y_t = z_t K_{t-1}^\alpha N_{t}^{1-\alpha}
\\
I_t = K_{t}  - \left(1 - \delta\right) K_{t-1} + \zeta \left(K_{t},K_{t-1}\right) K_{t-1}
\\
\psi^p_t =\frac{\mu_p}{\mu_p-1} \frac{1}{2 \kappa_p}\left[\log \left(1 + \pi_t\right)\right]^2 Y_t
\\
D_t = Y_t - w_t N_t - I_t - \psi^{p}_{t}
\\
w_t = (1-\alpha)\frac{Y_t}{N_t} mc_t
\\
\log(1+\pi_t) = \kappa_p \left(mc_t - \frac{1}{\mu_p} \right) + \frac{1}{1+r_{t+1}} \frac{Y_{t+1}}{Y_t} \log(1+\pi_{t+1})
\\
Q_t = 1 + \frac{1}{\delta \epsilon_I}\left(\frac{K_t-K_{t-1}}{K_{t-1}}\right)
\\
(1+r_{t+1})Q_{t} = \alpha \frac{Y_{t+1}}{K_t} mc_{t+1} - \left[\frac{K_{t+1}}{K_t} - (1-\delta) + \frac{1}{2\delta \epsilon_I}\left(\frac{K_{t+1} - K_t}{K_t}\right)^2\right] + \frac{K_{t+1}}{K_t}Q_{t+1}
\\
1 + \pi_t^w = (1 + \pi_t) w_t/w_{t-1}
\\
\log \left(1+\pi_t^w\right)=\kappa_w\left[\varphi N_t^{1+v}-\frac{\left(1-\tau_t\right) w_t N_t}{\mu_w} \int{e g_{c,t}^{-\sigma}d \Gamma_t}\right]+\beta \log \left(1+\pi_{t+1}^w\right)
\\
i_t = r_{\ast} + \phi \pi_t + \phi_y \left(Y_t - Y_{\ast}\right) + m_t
\\
1+r_t=\frac{1+i_{t-1}}{1+\pi_{t}}
\\
r_t^b = r_t - \omega
\\
\frac{D_{t+1} + p_{t+1}}{p_{t}} = 1 + r_{t+1}
\\
\Theta_{pt-1} = \frac{p_{t-1}}{p_{t-1} + B^{g} - B^{h}_{t-1}}
\\
1 + r_t^a = \Theta_{pt-1} \frac{D_t + p_t}{p_{t-1}} + (1 - \Theta_{pt-1}) \left(1 + r_t\right)
\\
\tau_t =\frac{r_{t}B^{g} + G_t}{w_t N_t}
\\
B^{h}_{t} = \int{g_{b,t} d \Gamma_t}
\\
p_{t} + B^{g}=\int{\left(g_{a,t} + g_{b,t}\right)d \Gamma_t}
$$

同时，商品市场出清条件隐含在瓦尔拉斯定律之中。该系统可以进一步简化。请参阅下面定义简化方程组的 hmod 文件。

### 均衡方程组：稳态均衡

在用常数 $\left(z_t, \pi_t, G_t\right) = \left(z_{\ast}, 0, G_{\ast}\right) $ 表示的稳态均衡中，均衡方程组可简化为：

$$
Y_{\ast} = z_{\ast} K_{\ast}^\alpha N_{\ast}^{1-\alpha}
\\
I_{\ast} = \delta K_{\ast}
\\
\psi^p_{\ast} = 0
\\
D_{\ast} = Y_{\ast} - w_{\ast} N_{\ast} - \delta K_{\ast}
\\
w_{\ast} = (1-\alpha)\frac{Y_{\ast}}{N_{\ast}} mc_{\ast}
\\
mc_{\ast} = \frac{1}{\mu_p}
\\
Q_{\ast} = 1
\\
r_{\ast} + \delta = \alpha \frac{Y_{\ast}}{K_{\ast}} mc_{\ast}
\\
\pi_{\ast}^w = 0
\\
\varphi N_{\ast}^{v} = \frac{\left(1-\tau_{\ast}\right) w_{\ast}}{\mu_w} \int{e g_{c,{\ast}}^{-\sigma}d \Gamma_{\ast}}
\\
i_{\ast} = r_{\ast}
\\
i_{\ast} = r_{\ast}
\\
r_{\ast}^b = r_{\ast} - \omega
\\
D_{\ast} = r_{\ast} p_{\ast}
\\
\Theta_{p{\ast}} = \frac{p_{\ast}}{p_{\ast} + B^{g} - B^{h}_{\ast}}
\\
r_{\ast}^a = r_{\ast}
\\
\tau_{\ast} =\frac{r_{\ast}B^{g} + G_{\ast}}{w_{\ast} N_{\ast}}
\\
B^{h}_{\ast} = \int{g_{b,{\ast}} d \Gamma_{\ast}}
\\
p_{\ast} + B^{g}=\int{\left(g_{a,{\ast}} + g_{b,{\ast}}\right)d \Gamma_{\ast}}
$$

## hmod 文件

模型可以由如下的 hmod 文件表示：


```{literalinclude} hank2_ssj.hmod
:language: HANS
```

## 使用工具箱

```{raw} html
:file: hank2_ssj_notebook.html
```

```{raw} latex
\includepdf[pages=-]{../../source/examples/HANK2_SSJ/hank2_ssj_notebook.pdf}
```