# Probably Approximately Correct Learning

在 ERM 规则下，对于一个有限假设类，，如果这个训练集的样本容量越大，那么输出的假设函数就越接近于正确情况，这种定义被称为 PAC 学习。

## Sample Complexity（样本复杂度）
函数 $m_{\mathcal{H}}:(0,1)^2 \rarr \mathbb{N}$  决定了学习假设类 $\mathcal{H}$ 的样本复杂度，也就是说要满足 PAC 方案所需要的样本数量。

样本复杂度是一个关于精确参数 $\epsilon$ 和 置信参数 $\delta$ 的函数。此外，这个函数还依赖于假设类 $\mathcal{H}$ 的性质。例如，对于一个有限假设类，其样本复杂度依赖于假设类容量的log函数值。

如果假设类 $\mathcal{H}$ 满足 PAC，那么对于给定的 PAC 定义就有很多满足需求的 $m_{\mathcal{H}}$ 函数。这里选择“最小函数”，也就是说对于任何精确参数 $\epsilon$ 和 置信参数 $\delta$，$m_{\mathcal{H}}(\epsilon, \delta)$ 选择能够满足 PAC 学习的最小整数。

**定义：任意一个有限假设类如果是 PAC 可学习的，那么样本复杂度满足：**
$$
m_{\mathcal{H}}(\epsilon, \delta) \leqslant 
\lceil
\frac{ \log(|\mathcal{H}|/\delta)}{\epsilon}
\rceil
$$

## 定义：PAC Learnability
如果存在：
一个函数：$m_{\mathcal{H}}:(0,1)^2 \rarr \mathbb{N}$ ；
一个学习算法满足以下特性：
1. 对于任意的 $\epsilon$
2. 对于任意的 $\delta \in (0,1)$
3. 对于 $\mathcal{X}$ 上的任意分布 $\mathcal{D}$
4. 对于任意的标签函数 $f:\mathcal{X} \rarr \{0, 1\}$

如果可学习性假设满足 $\mathcal{H, D},f$，并且算法在一个容量 $m \geqslant m_{\mathcal{H}}(\epsilon, \delta)$ ；由分布 $\mathcal{D}$ 独立同分布采样并且使用 $f$ 函数获取标签的训练集上计算，那么算法返回的假设函数 $h$ 时满足 PAC 可学习性的。


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwMDcyNjI4MDddfQ==
-->