# **HANS**: 含有异质性主体宏观模型的求解工具箱

**HANS** 是一个求解异质性主体（Heterogeneous-Agent, HA）宏观模型非线性解的工具箱，其算法基于文章 Bai, Luo 和 Wang (2023) 。现有其他求解 HA 模型的工具箱包括：

- [Econ-ARK](https://econ-ark.org/)

- [PHACT](https://github.com/gregkaplan/phact)

- [Sequence Space Jacobian](https://github.com/shade-econ/sequence-jacobian)

- [BASEforHANK](https://github.com/BASEforHANK)

**HANS** 的优势在于能够解决具有任意维度内生状态变量的 HA 模型，处理包括投资组合选择在内的多维连续选择变量，处理离散选择，并且求解非线性转换路径。要了解该工具箱的功能和用法，请参见下面的示例列表。

**HANS** 的优势还在于以类似 Dynare 的直观模型脚本文件为输入，编译 C++ 二进制文件以进行高性能计算，并提供用户友好的 MATLAB 接口。例如，Krusell 和 Smith (1998) 的模型可以用以下工具箱脚本表示：

```{literalinclude} examples/KS_leading/KS_leading.hmod
:language: HANS
:linenos:
```

请参照以下示例开始使用 **HANS**。

```{toctree}
:maxdepth: 2
:caption: Contents
   
Home <self>
examples/KS_JEDC10/ks.md
examples/HANK1_ZLB/hank1_zlb.md
examples/HANK2_SSJ/hank2_ssj.md
examples/KT2008/kt2008.md
cores/methodology.md
cores/api.md
```

