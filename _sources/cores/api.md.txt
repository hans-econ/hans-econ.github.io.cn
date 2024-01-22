(toolbox_api)=

# 工具箱调用接口说明（API Reference）

## 脚本 (hmod) 文件编辑说明

:::{important}
所有语句均需以英文输入法状态下的半角分号（;）结尾。
:::

```{py:function} parameters var1 var2 ...;

声明 **vfi** 模块中包含的参数。

"parameters" 需包含：（1）在 **vfi** 模块中的参数；（2）在 *var_pre_vfi* 模块中并随 *var_agg* 变化的参数。

```


````{py:function} var_shock var1 var2 ...;

声明个体外生状态变量，需为离散的马尔科夫链。

需在变量 *shock_trans* 中给出相应的伴随转移矩阵。

例子：

```HANS

var_shock e z;
e_grid = [1.0, 2.0];
e_trans = [0.4 0.6;0.6 0.4];
z_grid = [2.0,3.0,4.0];
z_trans = [0.6 0.2 0.2;0.2 0.6 0.2; 0.2 0.2 0.6];
% 生成张量组合
[e,z] = ndgrid(e,z);
e = e(:)';
z = z(:)';
shock_trans = kron(z_trans, e_trans);

```
````


```{py:function} var_state var1 var2 ...;

声明个体内生状态变量，需为连续变量。

需明确每个离散状态变量的格点。工具箱将使用线性插值法或三次样条插值法在格点上近似值函数与策略函数。

```


````{py:function} var_pre_vfi var1 var2 ...;

声明由 *var_shock* 或 *var_state* 定义的变量，这些变量需在声明 vfi 模块前被赋值。

在 *var_pre_vfi* 模块中声明的变量可被用于定义 *var_policy* 的初始值或直接在 vfi 模块中使用。这避免了在求解最优化问题过程中不必要的重复计算。

例子：

```{sourcecode} HANS
:linenos:
:emphasize-lines: 10,11,17

parameters beta r w;
...

var_shock e;
...

var_state a;
...

var_pre_vfi budget;
budget = (1+r)*a + w*e;

var_policy ap;
...

vfi;
    c = budget - ap;
    Tv = u(c) + beta*EXPECT(v(ap));
    ap >= 0.0;
end;

```
````


````{py:function} var_policy var1 var2 ...;


声明个体策略变量，即异质性主体优化的变量。

个体策略变量的初始值需给定，可由 *var_state* 和/或 *var_pre_vfi* 确定。

例子：

```{sourcecode} HANS
:linenos:
:emphasize-lines: 13,14,15

parameters beta r w;
...

var_shock e;
...

var_state a;
...

var_pre_vfi budget;
budget = (1+r)*a + w*e;

var_policy c ap;
initial ap 0.0;
initial c budget;

vfi;
    Tv = u(c) + beta*EXPECT(v(ap));
    ap >= 0.0;
    c + ap == budget;
end;

```

````


````{py:function} var_aux var1 var2 ...;


声明辅助变量，可表示为 *var_shock*，*var_state* 和/或 *var_policy* 的简单函数。

辅助变量的取值需在 **vfi** 模块中定义并返回。

例子：

```{sourcecode} HANS
:linenos:
:emphasize-lines: 17,23

parameters beta r w;
...

var_shock e;
...

var_state a;
...

var_pre_vfi budget;
budget = (1+r)*a + w*e;

var_policy c ap;
initial ap 0.0;
initial c budget;

var_aux inc;

vfi;
    Tv = u(c) + beta*EXPECT(v(ap));
    ap >= 0.0;
    c + ap == budget;
    inc = a*r + w*e;
end;

```
````


````{py:function} vfi; ... end;

定义 **vfi** 模块。此模块将从上到下、逐行运行。在此模块中：

- 每次迭代后的更新值需赋值给变量 “Tv”。
- 最优化的目标需在名为 “objective” 的变量中给出。如果没有变量 “objective”，则默认使用变量 “Tv” 作为最优化的目标。
- 用于下一次迭代的值函数（即上一次迭代得到的值函数）可通过调用 v(var_state_1_future,var_state_2_future,…) 来获得。如果函数调用被 EXPECT() 作用，则会基于当前外生状态所对应的转移矩阵计算下一期值函数的期望值。
- 等式或不等式约束可用 “==”、“<=” 或 “>=” 表示。
— 所有 *var_aux* 变量的定义都需在这个模块给出。
- 状态转移默认为评估未来值函数时的自变量。也可通过下述语句明确指定状态转移过程： *var_state*(+1) = {state_value_1, state_value_2，…} {prob_1, prob_2，}，其中，state_value_1、state_value_2 等为状态值，prob_1、prob_2 等为相关的状态转移概率。

例子：

```HANS

parameters beta r w;
...

var_shock e;
...

var_state a;
...

var_pre_vfi budget;
budget = (1+r)*a + w*e;

var_policy c ap;
initial ap 0.0;
initial c budget;

var_aux inc;

vfi;
    % 更新贝尔曼方程 Tv 的取值
    % Tv 也是最优化的目标
    % “ap” 会自动成为 a 的状态转移
    Tv = u(c) + beta*EXPECT(v(ap));

    % 也可明确指定状态转移过程
    a(+1) = {ap} {1};

    % 等式和不等式约束
    ap >= 0.0;
    c + ap == budget;

    % var_aux 的值
    inc = a*r + w*e;
end;

```
````

:::{note}
下述均衡模型的定义和相关信息是可选的。工具箱可以生成独立的代码仅用于求解值函数迭代。
:::

```{py:function} var_agg var1 var2 ...;
声明总体均衡条件对应的未知变量。

未知变量的初始值需给出。
```


```{py:function} var_agg_shock var1 var2 ...;
声明外生的，随时间变化的总体变量。

外生时变的总体变量的稳态值需给出。

对于命名为 “var” 的 *var_agg_shock*，其时间序列值可以通过在传递给模型求解脚本的选项中提供一个名为 “var_t” 的字段来覆盖。
```


````{py:function} model; ... end;
定义模型的总体均衡系统。在此模块中：

- 使用运算符“==”定义均衡方程。
- 代码从上到下计算，并可赋值给中间变量。
- 在定义方程组之前，需更新所有随 *var_agg* 变化的参数，如资本存量决定的的工资和利率。
- 在定义方程组时，可使用变量 *var_policy*，表示对个体策略函数的加总。
- 变量的滞后值和先导值用 varname(-l) 或 varname(+l) 表示，其中 l 为滞后或前导的期数。只有在 *var_agg* 中声明的变量才能使用其滞后值和先导值，即被赋值的中间变量不能使用滞后值和先导值进行索引。
- 预先确定的变量（如RBC中的资本）需要作为滞后变量进入系统。
- 方程求解后将返回所有在等式左边的赋值变量，便于计算其他感兴趣的均衡变量。

例子：

```HANS

parameters beta r w;
...


var_state a;
...

var_policy ap;
...

vfi;
	...
end;

var_agg K;
K = 30.0;	% 用于求解稳态的初始值
var_agg_shock Z;
Z = 1.0;	% 稳态值

model;
    % 更新进入 vfi 的参数
    % 资本是预先确定的，使用滞后值 K(-1)
    r = alpha * Z * K(-1)^(alpha-1) * L^(1-alpha) - delta;
    w = (1-alpha) * Z * K(-1)^alpha * L^(-alpha);

    % 使用 "==" 定义均衡方程
    % 下面使用的 ap 是 ap 在个体间的加总值
    K == ap; % 资产总需求 = 资产总供给

    % 后续赋值；所有被赋值的变量将被返回
    Y = Z*(K(-1)^alpha)*(L^(1-alpha));
    I = K - (1 - delta)*K(-1);
    C = Y - I;
end;

```
````

````{py:function} model_modname(var_agg_1, var_agg_2); ... end;
定义一个备选的均衡系统，其变量为 (var_agg_1, var_agg_2，…)。

解析器会为每个备选的 model_modname 模块生成一个名为 “solve_modname.m” 的脚本文件。

由此，不同的模型将共享相同的 **vfi** 模块。例如，用于校准的系统不同于求解均衡的系统。

例子：

```HANS

...

var_agg K;

model;
    % 此模块用于求解均衡
    r = alpha * Z * K(-1)^(alpha-1) * L^(1-alpha) - delta;
    w = (1-alpha) * Z * K(-1)^alpha * L^(-alpha);
    K == ap; % 资产总需求 = 资产总供给
end;

model_cali(K,beta);
    % 此模块用于校准 
    r = alpha * Z * K(-1)^(alpha-1) * L^(1-alpha) - delta;
    w = (1-alpha) * Z * K(-1)^alpha * L^(-alpha);
    Y = Z*(K(-1)^alpha)*(L^(1-alpha));
    K == ap; % 资产总需求 = 资产总供给
    K/Y == KY_ratio_target;	% 用于校准的目标方程
end;
```
````


## MATLAB 接口


```{py:function} solve_vfi(options)

值函数迭代求解。

:param options: 此结构可包含下述字段
	- interp_order: 用于值函数近似的内插与外插维度。默认值：'4+3'
		- '4': 三次样条插值
		- '2': 线性插值
		- '4+2': 三次内插 + 线性外插
		- '4+3': 三次内插 + 二次外插 
		- 'pchip': MATLAB 的 pchip 函数（仅内生状态变量为一维时适用）
    - algorithm: 用于求解最优化问题的算法。当求解一维优化问题时，默认值：'dbrent'；否则，默认值：“donlp2”。可选的值有:
    	- 'dbrent'
    	- 'golden'
    	- 'brent'
    	- 'nr3_dbrent'
    	- 'donlp2' 
	- vfi_tol_v: 值函数收敛的误差容忍度。默认值：1e-6
	- vfi_tol_policy: 策略函数收敛的误差容忍度。默认值：1e-12
	- print_freq: 输出迭代信息的频率。默认值：100
	- solver_tol_x: 个体优化问题的误差容忍度。默认值：1e-8
	- solver_tol_con: 个体优化问题的可行性容差。默认值：1e-8
	- num_threads: 求解个体优化问题时使用的线程数。默认值：由 'feature(numcores)' 返回的值

:param options: 已有的参数值会被此结构中包含的参数值覆盖。

:return: 此结构包含已解出的策略函数

```

%%

```{py:function} solve_ss(options)

求解 **model** 模块中定义的均衡系统的稳态值。

:param options: 此结构可包含下述字段
	- ss_solver: 用于求解稳态系统的方程求解器。默认值：'Broyden'。可选的值有:
		- 'Broyden'
		- 'Newton'
		- 'lsqnonlin'
		- 'CoDoSol'
		- 'fsolve'
    - ss_ftol: 传递给方程求解器的方程残差公差。默认值：1e-4
    - ss_xtol: 传递给方程求解器的步长公差。默认值：1e-6
    - ss_findiffstep: 计算数值导数的有限差分步长。默认值：1e-4
    - HANS_x: 未知变量的初始值

:param options: 还可包含需传递到值函数迭代（solve_vfi）的选项

:param options: 还可包含需被覆盖的参数值

:return: 此结构包含已解出的稳态值

```

%%

```{py:function} solve_trans_linear(ss, options)

求解线性化转移路径。利用虚拟消息算法（fake news algorithm）计算个体变量加总对于参数的雅可比矩阵。

:param ss: 此结构包含已解出的稳态值（由solve_ss返回），并在其附近进行线性化

:param options: 此结构可包含下述字段
	- T: 转移路径的期数。默认值：200

:param options: 对于名为 “var” 的 *var_agg_shock*，在此结构的 “var_t” 字段中提供更新的时间序列

:param options: 还可包含需传递到值函数迭代的选项

:param options: 还可包含需被覆盖的参数值

:return: 此结构包含已解出的转移路径和个体变量加总对于参数的雅可比矩阵

```

```{py:function} solve_trans_nonlinear(init_ss, final_ss, options)

求解非线性转移路径。

:param init_ss: 此结构包含转移路径开始时的初始稳态值

:param final_ss: 此结构包含转移路径收敛到的最终稳态值

:param options: 此结构可包含下述字段
	- T: 转移路径的期数。默认值：200
	- trans_linear_rslt: 已解出的线性化转移路径，包含个体变量加总对于参数的雅可比矩阵；将被用于构成非线性方程组的初始雅可比矩阵。默认值：empty，将调用 solve_trans_linear 计算
	- trans_solver: 方程求解器。默认值："Broyden"。可选的值有:
		- "Broyden"
		- "FixedJacobian"
		- "BroydenMod"
		- "Newton"
		- "fsolve"
    - trans_ftol: 传递给方程求解器的方程残差公差。默认值：1e-4
    - trans_xtol: 传递给方程求解器的步长公差。默认值：1e-6
    - HANS_x: 未知变量的初始值

:param options:还可包含需传递到值函数迭代的选项

:param options: 还可包含需被覆盖的参数值

:return: 此结构包含已解出的非线性转移路径

```

%%

```{py:function} solve_*modname*(options)
求解 model_*modname* 模块中定义的备选均衡系统的稳态值。

此函数的输入和输出结构与 solve_ss 相同。
```

%%

```{py:function} simulate_stochastic(eq_rslt, options)

用已解出的均衡来模拟个体随机样本。

:param eq_rslt: 此结构包含稳态均衡（由 solve_ss 返回）或转移路径（由 solve_nonlinear_trans 返回）

:param options: 此结构可包含下述字段
	- num_samples: 个体样本数。默认值：1e4
	- num_periods: 时期数。默认值：1e3
	- simulate_initials: 此结构包含状态变量的初始值。默认值：empty，将从稳态分布中对状态值进行抽样
	- interp_order: 策略函数近似的内插与外插维度。默认值：'2'
		- '4': 三次样条插值
		- '2': 线性插值
		- '4+2': 三次内插 + 线性外插
		- '4+3': 三次内插 + 二次外插 
	- simu_print_freq: 输出模拟统计量的频率。默认值：1000
	- num_threads: 用于策略函数插值的线程数。默认值：由 'feature(numcores)' 返回的值
```