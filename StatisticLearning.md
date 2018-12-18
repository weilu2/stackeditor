
## 2.2. Empirical Risk Minimization（经验风险最小化）

**定义**
$S$ 是一个从全体数据集中采样得到的一个训练集；
$\mathcal{D}$ 是采集训练集 $S$ 时所服从的一种未知的分布；
$f$ 是一个目标函数，给训练集中的每个样本打上标签；
$h_S: \mathcal{X} \rarr \mathcal{Y}$ 是一个预测函数，这个函数用来预测给定输入 $\mathcal{X}$ 到输出 $\mathcal{Y}$；

算法目标：找到一个 $h_S$ 能够最接近的表示这个未知的分布 $\mathcal{D}$ 和函数 $f$；

$L_S(h)$ 表示预测函数和实际情况的误差。但是我们并不知道实际上这个未知的分布 $\mathcal{D}$ 和函数 $f$ 到底是多少，因此我们只能计算预测函数和训练集的偏差：
$$
L_S(h) \stackrel{\text{def}}{=} \frac{|\{i \in [m]:h(x_i)\ne y_i \}|}{m}, [m] = \{1, \ldots, m\}
$$

**empirical error** 或 **empirical risk** 指的就是这个 $L_S(h)$。

**Empirical Risk Minimization, ERM** ：通过使得经验风险 $L_S(h)$ 最小，而找到的一个预测函数 $h_S$ 的这个学习范式被称为经验风险最小。

**解释**
我们所处的这个世界上任何一项活动所产生的所有数据 $T$，是没有办法全部收集到的。在这种情况下，我们只能收集到一些有限的数据 $S$，同时也从实际环境中收集到了这些数据对应的标签 $L$。

但我们并不知道这些收集到的数据 $S$ 在实际的数据集 $T$ 中的分布 $\mathcal{D}$ 是如何的，并且我们也不知道从 $S$ 到 $L$ 的这个关系 $f$ 是如何的。

所以我们只能去猜测某种关系 $h_S$，用这个关系去近似的表示数据集 $S$ 到 $L$ 的关系，用 $L_S(h)$ 来表示关系 $h_S$ 和实际 $S$ 到 $L$ 关系的误差，通过使得这个误差最小，就能够找到一个能够对这些数据进行预测的函数。

这种寻找这个预测函数的学习范式被称为经验风险最小化。

### 2.2.1. overfittting（过拟合）
当 $L_S(h)$ 在数据集 $S$ 上非常小，$h_S$ 在 $S$ 上表现的非常好。但是在对不属于训练集中的数据预测时效果很差，这种情况被称为过拟合。

这种情况非常符合经验风险最小化原则，但是却不是理想的预测函数，这是经验风险最小化范式的一个问题。

## 2.3. 使用归纳偏差的经验风险最小化

修正 ERM 学习规则容易导致过拟合问题的一个方案是在一个受限的搜索空间中应用 ERM 学习规则。

$\mathcal{H}$ 称作 **hypothesis class（假设类）** ，表示一个预测函数的集合。其中的每个元素 $h \in \mathcal{H}$ 表示一个从 $\mathcal{X}$ 到 $\mathcal{Y}$ 的映射函数。

对于给定的假设类 $\mathcal{H}$ 和训练集 $S$，运用 ERM 规则通过最小化预测误差 $L_S(h)$ 来从假设类中选择一个预测函数 $h \in \mathcal{H}$ 表示为：
$$
ERM_{\mathcal{H}}(S) \in \arg \min_{h \in \mathcal{H}} L_S(h)
$$
其中，$\arg \min$ 表示假设类 $\mathcal{H}$ 中，$L_S(h)$ 最小的假设函数的集合。

**inductive bias（归纳偏置）** 通过限定从 $\mathcal{H}$ 中选择预测函数，在选择假设函数时会偏向于一个特定的预测函数集合，我们称之为归纳偏置。

因此，选择假设类对于能否选择合适的预测函数至关重要。下面我们看如何选择假设函数类。

对于一个给定的训练样本 $S$，使用 $h_S$ 表示对$S$ 应用 $ERM_H$ 之后得到的结果：
$$
h_S \in \arg \min_{h \in \mathcal{H}} L_S(h)
$$
我们假设，存在 $h^* \in \mathcal{H}$，使得 $L_{\mathcal{D},f} (h^*) = 0$。这个假设意味着存在一个函数 $h^*$ 对于任意按照分布 $\mathcal{D}$ 采集的随机样本 $S$，并使用 $f$ 函数得到标签的数据都有 $L_S(h^*) = 0$。

虽然对于每个 ERM 假设类都存在能够使得 $L_S(h_S) = 0$ 的假设函数，但是我们真正关注的是基于实际的分布和标签函数的误差 $L_{(\mathcal{D},f)}(h_S)$，而不是经验风险。

对于任何一个只能访问样本 $S$ 数据的算法而言，其能否相对准确的表示潜在的分布 $\mathcal{D}$，依赖于样本 $S$ 和分布 $\mathcal{D}$ 之间的关系。一般假设样本中的每个元素都是根据分布 $\mathcal{D}$ 独立的获取的。

**The i.i.d. assumption（独立同分布假设）**：训练集中的每个数据都是独立的以相同的分布 $\mathcal{D}$ 进行采集的。我们将这个假设表示为 $S \sim \mathcal{D}^m$，其中 $m$ 表示训练集 $S$ 的容量，$\mathcal{D}^m$ 表示抽取 $m$ 个样本的概率。

我们可以将训练集 $S$ 看做是一个描述真实世界的视窗，如果这个视窗越大，那么我们能够看到的真实世界的分布 $\mathcal{D}$ 也就越准确，同理其生成标签的 $f$ 也看的越准确。

$L_{(\mathcal{D},f)}(h_S)$ 依赖于训练集 $S$，而训练集的采集过程是一个随机过程，因此预测函数 $h_S$ 的选择就有一定的随机性，那么同样的经验风险  $L_{(\mathcal{D},f)}(h_S)$ 也同样的有一定的随机性。

**confidence parameter（置信参数）**：一般来说，对于一个训练集 $S$ ，其中的每个元素都能够正好反应出实际的分布 $\mathcal{D}$ 是不现实的。通常都有一定的概率采集到的样本不能很好的表示潜在的分布 $\mathcal{D}$。我们使用 $\delta$ 表示采集到一个不具有代表性样本的概率，并将 $1-\delta$ 称之为预测函数的置信参数。

**accuracy parameter（精度参数）**：由于我们没有办法百分之百准确的预测标签，因此引入一个变量 $\epsilon$，只要对某个样本的某个标签预测的误差大于这个变量 $L_{(\mathcal{D},f)}(h_S) >\epsilon$，那么就认为预测结果不是这个标签；如果对某个样本的某个标签预测的误差小于等于这个变量 $L_{(\mathcal{D},f)}(h_S) \leqslant \epsilon$，就认为这个样本属于这个标签。

训练集的实例：
$$
S|_x = (x_1, \ldots, x_m)
$$

使用下面的式子表示对于一个有 $m$ 个样本的训练集，预测失败的概率的上界：
$$
\mathcal{D}^m ( \{ S|_x: L_{(\mathcal{D},f)}(h_S) > \epsilon \})
$$
使用下面式子表示不好的假设函数的集合：
$$
\mathcal{H}_B = \{ h \in \mathcal{H} : L_{(\mathcal{D},f)}(h) > \epsilon \}
$$

使用下面的式子表示误导样本集合：
$$
M = \{ S|_x: \exist h \in \mathcal{H}_B, L_S(h) = 0 \}
$$

对于 $M$ 集合中的每个元素都有一个坏的假设函数，由于这个假设函数的损失 $L_S(h) = 0$，看起来是一个好的假设函数，但由于 $L_{(\mathcal{D},f)}(h) > \epsilon$，因此实际上是个坏函数。集合 $Ｍ$ 可以表示为：
$$
\{ S|_x:L_{(\mathcal{D},f)}(h) > \epsilon \} \sube M
$$
还可以将 $M$ 写成：
$$
M = \bigcup_{h \in \mathcal{H}_B} \{ S |_x : L_S(h) = 0 \}
$$

因此：
$$
\mathcal{D}^m ( \{ S|_x: L_{(\mathcal{D},f)}(h_S) > \epsilon \}) 
\leqslant \mathcal{D}^m(M)=
\mathcal{D}^m(\bigcup_{h \in \mathcal{H}_B} \{ S |_x : L_S(h) = 0 \})
$$

对等号右边的式子利用**union bound（联合界）**公式：
$$
\mathcal{D}(A \cup B) \leqslant \mathcal{D}(A) + \mathcal{D}(B)
$$
可以转换为：
$$
\mathcal{D}^m ( \{ S|_x: L_{(\mathcal{D},f)}(h_S) > \epsilon \}) 
\leqslant 
\sum_{{h \in \mathcal{H}_B}} \mathcal{D}^m(\{ S |_x : L_S(h) = 0 \})
$$
由于事件 $L_S(h) = 0$ 等同于 $\forall i, h(x_i) = f(x_i)$，因此上面式子中的求和公司的每一项都可以表示为：
$$
\mathcal{D}^m(\{ S |_x : L_S(h) = 0 \}) = \mathcal{D}(\{ S |_x : \forall i, h(x_i) = f(x_i) \})
\\ \quad\quad\quad\quad \quad \quad\quad\quad\quad\space= 
\prod_{i = 1}^m \mathcal{D}(\{ x_i : h(x_i) = f(x_i) \})
$$

对于训练集中的每个独立样本，可以表示为：
$$
\mathcal{D}(\{ x_i : h(x_i) = f(x_i) \}) = 1 - L_{\mathcal{D},f}(h) \leqslant 1 - \epsilon
$$

根据不等式 $1 - \epsilon \leqslant e^{-\epsilon}$，可以得到：
$$
\mathcal{D}(\{ x_i : h(x_i) = f(x_i) \})  \leqslant 1 - \epsilon \leqslant e^{-\epsilon}
$$

根据上面的式子，可以将这个式子转换为：
$$
\mathcal{D}^m(\{ S |_x : L_S(h) = 0 \}) \leqslant (1 - \epsilon)^m \leqslant e^{-m\epsilon}
$$

结合根据联合界得到的式子，可以得到：
$$

$$
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcxNDMzODA2Ml19
-->