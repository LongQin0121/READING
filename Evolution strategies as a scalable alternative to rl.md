
# Evolution Strategies as a Scalable Alternative to Reinforcement Learning
## OpenAI 团队
这篇文章的重点之一就是说明 ES 在计算速度上的优势，主要原因是：

高度可并行（Highly Parallelizable）

ES 每次迭代需要做的事情是：对策略加上很多随机扰动，然后分别在环境中跑一遍、算奖励。

这些扰动策略之间 互不依赖，所以可以同时分配到成百上千个 CPU/GPU 上去跑。

结果是：资源越多，速度几乎可以 线性提升。

不需要反向传播（No Backpropagation）

传统 RL（比如 DQN、PPO）需要反向传播来更新参数，涉及梯度计算和复杂的优化过程。

ES 只需要前向运行策略 → 得到奖励 → 聚合结果，不用算梯度，省掉了很多计算。

实现简单，通信开销小

并行节点之间只需要传递一个“适应度值”（奖励标量），而不是整条梯度信息或经验回放池，通信成本更低。

所以文章想表达的是：
👉 ES 的计算效率很高，特别是在大规模分布式环境里，它可以比传统 RL 更快


二、代表性后续研究 / 改进工作

以下是几篇在 ES + RL 交叉领域里比较有代表性、被社区关注的后续工作或改进方向，我为每篇简单摘要关键贡献和它们与原文的关系：

论文 / 工作	关键贡献 / 新思路	与原文（Salimans et al. 2017）的关系 / 优化方向
Improving Exploration in Evolution Strategies for Deep Reinforcement Learning via a Population of Novelty-Seeking Agents （Conti et al., NeurIPS 2018）	融入 novelty search（新颖性搜索） 的思想，用一部分个体探索“多样性”而不仅仅追求奖励，以应对稀疏奖励或容易陷入局部最优的问题。该方法被称为 NS-ES / QD-ES。 
NeurIPS Papers
	这是对 OpenAI-ES 的一个扩展，专注于 探索 性能改进，是原方法在某些环境下的有力补充。
Importance Weighted Evolution Strategies (IW-ES)	通过 重要性采样（Importance Sampling） 重用已有的经验批次，以提升样本利用效率（即减少浪费经验）。 
arXiv
	OpenAI-ES 本身每次 perturbation 生成的批次用一次就丢掉，IW-ES 尝试在保持并行性优势的同时，改进数据效率。
Instance Weighted Incremental Evolution Strategies for Reinforcement Learning in Dynamic Environments	针对环境可能变化（非静态环境）的场景，引入“实例加权”（instance weighting）机制，让以往经验中更能体现“新知识”的样本拥有更高权重，从而更快适应变化。 
arXiv
	这是对 ES 在变化环境下的适应性增强。原文主要假设环境相对静态。
ERL-Re²: Efficient Evolutionary Reinforcement Learning with Shared State Representation and Individual Policy Representation	探索将 RL 和进化算法结合：共享高阶的状态表示（representation），而每个个体有自己的策略表示（policy 表示），以便在演化过程中进行变异／交叉，同时仍借助 RL 的表示学习优势。 
arXiv
	这种混合方法是往 “融合 RL + ES 优点” 的方向走，是原文思想的一个延伸融合。
Proximal evolutionary strategy: improving deep reinforcement（2024 年一篇最新文章）	引入类似 “Proximal” 的机制来稳定更新、控制步长等，从 RL 的类比手法中汲取灵感，以改进 ES 的稳定性与效率。 
SpringerLink
	是较新的尝试，把 ES 的随机搜索和 RL 中的稳定更新机制相结合，旨在让 ES 在更多任务上更可靠。
Hierarchical RL + ES：Scalable ES as a hierarchical RL 方法	例如 “Evolution Strategies as an Alternate Learning method for Hierarchical Reinforcement Learning” 这篇论文，探讨将 ES 用于分层 RL（HRL）框架，从而处理有长时程决策层级的问题。 
OpenReview
	这是把 ES 引入 RL 更复杂结构（比如层级结构）的一种尝试，扩展 ES 能处理的问题种类。

此外，还有不少综述／对比文章把这篇 OpenAI-ES 作为参考基线，讨论其优缺点、适用场景等，比如：

Deep Reinforcement Learning Versus Evolution Strategies: A Comparative Survey —— 它将 RL 和 ES 方法在多个维度上进行比较，讨论优劣、适用条件等。 
arXiv

Evolutionary Reinforcement Learning: A Systematic Review（2024）—— 它把过去在 EvoRL（进化 + 强化学习融合）方向上的诸多工作整理、分类，对 ES 在 RL 领域的作用、挑战和未来方向做了总结。 
arXiv



好问题 👍
你想知道这些后续工作主要改进的是 速度（计算效率）、样本效率（数据利用）、还是 探索能力 / 稳定性。我帮你逐一总结：

代表性后续研究 & 改进点

Conti et al. 2018 – Novelty Search + ES (NS-ES / QD-ES)

改进点：探索能力

原始 ES 只追求奖励，容易陷入局部最优（特别是稀疏奖励任务）。

他们加了 “新颖性奖励”，鼓励策略探索更多不同的状态空间。

**结果：**探索能力更强，更适合复杂 / 稀疏奖励环境。

Importance Weighted ES (IW-ES)

改进点：样本效率

原始 ES 每次生成的扰动样本只用一次就丢掉。

IW-ES 用 重要性采样 重用旧样本，让已有数据还能贡献梯度估计。

**结果：**减少样本浪费，在数据昂贵或不能超大规模并行时更高效。

Instance Weighted Incremental ES（动态环境版 ES）

改进点：适应性（样本利用率 + 环境变化处理）

针对环境会变化的情况（比如动态任务），过去的经验并非完全无用。

他们用“加权机制”让更相关的旧经验仍能提供指导。

**结果：**在动态环境下收敛更快，避免每次都“从零学起”。

ERL-Re²（Evolutionary Reinforcement Learning with Shared Representation）

改进点：样本效率 + 表示学习能力

结合 RL 的表示学习 和 ES 的并行优化。

共享一个全局的状态表示，每个个体只进化策略部分。

**结果：**学习速度更快，训练更稳定，同时利用了 RL 和 ES 的优势。

Proximal ES（2024 提出）

改进点：稳定性 + 训练效率

借鉴 RL 里 PPO 的思想（限制更新幅度），防止参数更新太大导致不稳定。

**结果：**更新更稳健，提升收敛速度，减少训练波动。

总体对比总结

速度（并行计算加速） → 原始 OpenAI-ES 已经很强，后续论文大多没再专门改“速度”，而是侧重别的方面。

样本效率（减少浪费） → IW-ES、ERL-Re²、动态环境 ES。

探索能力 → Novelty Search + ES。

稳定性 / 收敛性 → Proximal ES、混合方法（RL + ES）。

👉 可以理解为：
OpenAI-ES 最大优势是 速度（并行可扩展性），但劣势是 样本效率不高、探索弱、更新不稳。
后续的工作主要就是针对这些劣势做改进。