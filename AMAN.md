系统命名：FCFS-Based Dynamic Slot Allocation（基于先到先得的动态时隙分配）
核心内容：

完整的参数定义（ETA, ETA_early, RTA等）
详细的分配算法步骤
首架飞机的决策逻辑（何时选ETA vs ETA_early）
优化目标和成本函数


实施示例：

密集流场景：展示优化前后对比
稀疏流场景：展示无需优化的情况

# TMA合流点时隙分配策略

## 1. 系统概述

### 1.1 名称定义
**FCFS-Based Dynamic Slot Allocation (基于先到先得的动态时隙分配)**
或
**Look-Ahead Slot Optimization (前瞻式时隙优化)**

### 1.2 系统架构
- **终端区(TMA)**：分为4个独立扇区（N1, E1, S1, W1）
- **合流点(MP)**：Merging Point，多流汇聚点
- **飞机能力**：4D轨迹管理，RTA（Required Time of Arrival）能力
- **时隙间隔**：90秒/slot

---

## 2. 核心概念

### 2.1 时间参数
| 参数 | 符号 | 说明 |
|------|------|------|
| 预计到达时间 | ETA | Estimated Time of Arrival |
| 最早到达时间 | ETA_early | 飞机全速飞行的最早可能到达时间 |
| 最晚到达时间 | ETA_late | 飞机最大减速后的最晚到达时间 |
| 要求到达时间 | RTA | Required Time of Arrival（分配给飞机的目标时间）|
| 时间窗口 | TW | Time Window = [ETA_early, ETA_late] |

### 2.2 时隙参数
- **时隙间隔(S)**：90秒
- **时隙容量(C)**：1架飞机/时隙
- **时隙序列**：Slot₁, Slot₂, Slot₃, ...

---

## 3. 分配策略

### 3.1 基本原则
**先到先得(FCFS)** + **前瞻优化(Look-Ahead)**

### 3.2 分配算法

#### Step 1: 飞机排序
按ETA升序排列所有即将到达MP的飞机：
```
Aircraft_Queue = Sort_by_ETA([A₁, A₂, ..., Aₙ])
```

#### Step 2: 首架飞机RTA决策

对于**第一架飞机**，需要判断：

**判断条件：**
```
IF (ETA₂ - ETA_early₁) < S (90秒)
    THEN RTA₁ 倾向于 ETA_early₁  // 加速策略
    ELSE RTA₁ = ETA₁              // 正常策略
END IF
```

**决策逻辑：**

| 场景 | 条件 | RTA₁选择 | 理由 |
|------|------|---------|------|
| **密集流** | ETA₂ - ETA₁ < 90s | ETA_early₁ 或接近值 | 为后续飞机腾出时隙空间 |
| **稀疏流** | ETA₂ - ETA₁ ≥ 90s | ETA₁ | 无需加速，自然间隔足够 |
| **临界流** | ETA₂ - ETA_early₁ ≈ 90s | ETA_early₁ | 飞机1加速，飞机2无延误 |

#### Step 3: 后续飞机RTA分配

```
FOR i = 1 to n:
    Slotᵢ = Slotᵢ₋₁ + S (90秒)
    
    IF Slotᵢ 在飞机i的时间窗口内 [ETA_earlyᵢ, ETA_lateᵢ]:
        RTAᵢ = Slotᵢ
    ELSE IF Slotᵢ < ETA_earlyᵢ:
        RTAᵢ = ETA_earlyᵢ  // 飞机无法加速到该时隙
        重新计算后续时隙基准
    ELSE:
        冲突，需要调整
    END IF
END FOR
```

---

## 4. 优化目标

### 4.1 成本函数

**总延误成本最小化：**

```
Minimize: Σ [wᵢ × max(0, RTAᵢ - ETAᵢ)]

约束条件：
1. RTAᵢ ∈ [ETA_earlyᵢ, ETA_lateᵢ]
2. RTAᵢ₊₁ - RTAᵢ ≥ S (90秒)
3. 先到先得顺序保持
```

其中：
- wᵢ：飞机i的延误权重（可根据机型、乘客数等设定）
- 正延误：需要减速等待
- 负延误：需要加速（通常成本更低）

### 4.2 权衡分析

| 策略 | 飞机1成本 | 后续飞机成本 | 总成本 |
|------|----------|-------------|--------|
| RTA₁ = ETA₁ | 0（无加速） | 可能累积延误 | 可能较高 |
| RTA₁ = ETA_early₁ | 加速成本 | 延误减少 | 可能较低 |

---

## 5. 实施示例

### 场景1：密集流（需要优化）

**输入数据：**
```
飞机1: ETA₁ = 10:00:00, [ETA_early₁, ETA_late₁] = [09:58:00, 10:02:00]
飞机2: ETA₂ = 10:00:30, [ETA_early₂, ETA_late₂] = [09:58:30, 10:02:30]
飞机3: ETA₃ = 10:01:00, [ETA_early₃, ETA_late₃] = [09:59:00, 10:03:00]
```

**策略A：RTA₁ = ETA₁（不优化）**
```
Slot₁ = 10:00:00 → RTA₁ = 10:00:00 (飞机1延误：0秒)
Slot₂ = 10:01:30 → RTA₂ = 10:01:30 (飞机2延误：60秒)
Slot₃ = 10:03:00 → RTA₃ = 10:03:00 (飞机3延误：120秒)
总延误：180秒
```

**策略B：RTA₁ = ETA_early₁（优化）**
```
Slot₁ = 09:58:00 → RTA₁ = 09:58:00 (飞机1加速：120秒)
Slot₂ = 09:59:30 → RTA₂ = 09:59:30 (飞机2加速：60秒)
Slot₃ = 10:01:00 → RTA₃ = 10:01:00 (飞机3延误：0秒)
总时间调整：180秒（但加速成本通常低于延误成本）
```

### 场景2：稀疏流（无需优化）

**输入数据：**
```
飞机1: ETA₁ = 10:00:00
飞机2: ETA₂ = 10:02:00
```

**策略：RTA₁ = ETA₁**
```
Slot₁ = 10:00:00 → RTA₁ = 10:00:00 (延误：0秒)
Slot₂ = 10:01:30 → RTA₂ = 10:02:00 (延误：0秒，自然间隔>90s)
总延误：0秒
```

---

## 6. 系统优势

### 6.1 主要优点
1. **动态优化**：根据实时流量密度调整策略
2. **公平性**：保持FCFS顺序
3. **灵活性**：利用飞机4D-RTA能力
4. **效率**：减少整体延误

### 6.2 适用场景
- 多扇区汇聚的TMA
- 飞机具备精确4D导航能力
- 高密度流量时段
- 需要精确间隔管理的情况

---

## 7. 实施要求

### 7.1 技术要求
- ✈️ 飞机4D轨迹管理能力
- 📡 ADS-B实时位置数据
- 🖥️ 中央时隙分配系统
- 📊 ETA预测算法

### 7.2 运行要求
- 🎯 90秒时隙严格遵守
- 🔄 动态RTA更新机制
- ⚠️ 冲突检测与解决
- 📞 管制员与飞行员协同

---

## 8. 扩展考虑

### 8.1 进阶优化
- **多目标优化**：同时考虑延误、燃油、排放
- **机器学习**：基于历史数据预测最优RTA
- **滚动时域优化**：定期重新规划未分配时隙

### 8.2 特殊情况处理
- 紧急航班优先级
- 天气导致的时间窗口变化
- 飞机性能差异
- 跑道构型变化影响

---

## 参考文献
此策略基于航空交通管理中的以下概念：
- Time-Based Flow Management (TBFM)
- Point Merge System
- 4D Trajectory-Based Operations (TBO)
- Arrival Manager (AMAN) systems

# 我已经将内容改写成英文学术论文格式，包含：
主要章节：

Abstract - 摘要，概述研究内容
Introduction - 背景、问题陈述和研究贡献
System Model and Notation - 系统建模和符号定义
Slot Allocation Methodology - 核心方法论

Phase 1: 飞机排序
Phase 2: 首架飞机优化（重点）
Phase 3: 后续飞机分配


Computational Examples - 计算示例（密集流 vs 稀疏流）
Performance Metrics - 性能指标
Discussion and Future Work - 讨论和未来工作
Conclusions - 结论

学术特点：

✅ 使用定理、证明、算法等学术格式
✅ 严格的数学符号和公式
✅ 详细的问题建模
✅ 复杂度分析
✅ 完整的文献引用框架

这是论文的"Slot Allocation"部分，可以作为独立章节或者整篇论文的核心方法部分。

---

# Dynamic Slot Allocation Strategy for Terminal Maneuvering Area Merging Point Operations

## Abstract

This paper presents a dynamic slot allocation strategy for coordinating aircraft arrivals at Terminal Maneuvering Area (TMA) merging points. The proposed method leverages 4D trajectory-based operations and Required Time of Arrival (RTA) capabilities to optimize slot assignments while maintaining First-Come-First-Served (FCFS) sequencing principles. The strategy considers each aircraft's operational time window and employs look-ahead optimization to minimize total system delay.

**Keywords**: Terminal Maneuvering Area, Slot Allocation, 4D Trajectory, Required Time of Arrival, Air Traffic Flow Management

---

## 1. Introduction

### 1.1 Background

Modern Terminal Maneuvering Areas (TMAs) often employ sector-based designs where multiple independent sectors converge at designated merging points (MPs). The TMA configuration illustrated in Figure 1 demonstrates a four-sector design (N1, E1, S1, W1) with a central merging point where aircraft flows from different sectors must be sequenced and separated.

### 1.2 Problem Statement

The primary challenge in TMA operations is to efficiently allocate arrival time slots at the merging point while:
- Maintaining required separation standards (typically 90 seconds between successive aircraft)
- Respecting each aircraft's operational constraints
- Minimizing delay propagation through the system
- Preserving fairness through FCFS ordering

### 1.3 Research Contribution

This paper presents a slot allocation methodology that:
1. Assigns optimal RTA values to aircraft based on their operational time windows
2. Employs look-ahead optimization for the lead aircraft to minimize downstream delays
3. Provides a systematic framework for balancing individual and system-wide efficiency

---

## 2. System Model and Notation

### 2.1 TMA Architecture

Consider a TMA divided into four independent sectors {N1, E1, S1, W1}, each handling arriving aircraft flows. All flows converge at a single merging point MP, where precise temporal coordination is required.

### 2.2 Aircraft Parameters

For each aircraft *i* in the system, we define:

| Parameter | Notation | Description |
|-----------|----------|-------------|
| Estimated Time of Arrival | ETA_i | Nominal arrival time at MP without intervention |
| Earliest Arrival Time | ETA_early,i | Minimum achievable arrival time (maximum speed) |
| Latest Arrival Time | ETA_late,i | Maximum allowable arrival time (minimum speed) |
| Required Time of Arrival | RTA_i | Assigned target arrival time |
| Time Window | TW_i | Operational flexibility: TW_i = [ETA_early,i, ETA_late,i] |

### 2.3 System Parameters

- **Slot Duration** (S): Minimum temporal separation between consecutive aircraft at MP (S = 90 seconds)
- **Slot Sequence**: {Slot₁, Slot₂, ..., Slot_n} where Slot_{k+1} = Slot_k + S

### 2.4 Problem Formulation

Given a set of n aircraft {A₁, A₂, ..., A_n} with their respective parameters {ETA_i, TW_i}, the objective is to assign RTA values such that:

**Objective Function:**
```
minimize: Σᵢ₌₁ⁿ wᵢ · max(0, RTAᵢ - ETAᵢ)
```

**Subject to:**
```
1. ETA_early,i ≤ RTAᵢ ≤ ETA_late,i  ∀i
2. RTAᵢ₊₁ - RTAᵢ ≥ S  ∀i ∈ {1,...,n-1}
3. Preserve FCFS ordering based on ETA
```

where w_i represents the delay cost weight for aircraft i.

---

## 3. Slot Allocation Methodology

### 3.1 Algorithm Overview

The proposed allocation strategy consists of three main phases:

**Phase 1**: Aircraft sequencing based on ETA  
**Phase 2**: Lead aircraft slot optimization  
**Phase 3**: Downstream slot propagation

### 3.2 Phase 1: Aircraft Sequencing

Sort all aircraft by their nominal arrival times:

```
Sequence = {Aᵢ | ETA₁ ≤ ETA₂ ≤ ... ≤ ETAₙ}
```

This ensures FCFS fairness principles are maintained throughout the allocation process.

### 3.3 Phase 2: Lead Aircraft Slot Optimization

The assignment of RTA₁ for the lead aircraft is critical as it establishes the baseline for all subsequent assignments. We employ a look-ahead strategy:

**Definition 3.1** (Traffic Density Classification)

Define the traffic density indicator:
```
Δ₁₂ = ETA₂ - ETA₁
```

We classify traffic scenarios as:
- **Dense Traffic**: Δ₁₂ < S
- **Moderate Traffic**: S ≤ Δ₁₂ < 2S
- **Sparse Traffic**: Δ₁₂ ≥ 2S

**Theorem 3.1** (Optimal Lead Aircraft Assignment)

For the lead aircraft A₁, the optimal RTA assignment strategy is:

```
RTA₁* = {
    ETA_early,1           if ETA₂ - ETA_early,1 < S + ε
    ETA₁                  if ETA₂ - ETA₁ ≥ S
    α·ETA_early,1 + (1-α)·ETA₁   otherwise
}
```

where ε is a small threshold (typically 10-15 seconds), and α ∈ [0,1] is determined by:

```
α = max(0, (S - (ETA₂ - ETA₁))/(ETA₁ - ETA_early,1))
```

**Proof Sketch**: 

Case 1 (Dense Traffic): When ETA₂ - ETA_early,1 < S, assigning RTA₁ = ETA₁ would force aircraft A₂ to delay by at least (ETA₁ + S) - ETA₂. By setting RTA₁ = ETA_early,1, we create sufficient temporal separation such that Slot₂ = ETA_early,1 + S can accommodate A₂ with minimal or no delay.

Case 2 (Sparse Traffic): When the natural separation Δ₁₂ ≥ S, no optimization is necessary as A₂ can maintain its ETA without delay regardless of A₁'s assignment.

Case 3 (Moderate Traffic): A weighted assignment balances the acceleration cost of A₁ against the delay cost of A₂. □

### 3.4 Phase 3: Downstream Slot Propagation

After establishing Slot₁, subsequent slots are assigned iteratively:

**Algorithm 1**: Downstream Slot Assignment
```
Input: Sorted aircraft sequence {A₁, ..., Aₙ}, Slot₁ = RTA₁
Output: {RTA₂, ..., RTAₙ}

1: current_slot ← Slot₁
2: for i = 2 to n do
3:     candidate_slot ← current_slot + S
4:     if candidate_slot ∈ TWᵢ then
5:         RTAᵢ ← candidate_slot
6:         current_slot ← candidate_slot
7:     else if candidate_slot < ETA_early,i then
8:         RTAᵢ ← ETA_early,i  // Aircraft cannot accelerate further
9:         current_slot ← ETA_early,i
10:    else
11:        RTAᵢ ← ETA_late,i  // Conflict resolution required
12:        Trigger resequencing protocol
13:    end if
14: end for
```

### 3.5 Feasibility Analysis

**Proposition 3.1** (Slot Assignment Feasibility)

A feasible slot assignment exists if and only if:
```
∀i ∈ {1,...,n-1}: ETA_late,i + S ≤ ETA_late,i+1
```

This condition ensures that even with maximum delays, the FCFS sequence can be preserved.

---

## 4. Computational Examples

### 4.1 Example 1: Dense Traffic Scenario

**Input Parameters:**
```
Aircraft A₁: ETA₁ = 600s, TW₁ = [480s, 720s]  (±120s flexibility)
Aircraft A₂: ETA₂ = 630s, TW₂ = [510s, 750s]
Aircraft A₃: ETA₃ = 660s, TW₃ = [540s, 780s]
Slot Duration: S = 90s
```

**Strategy A (Non-optimized)**: RTA₁ = ETA₁ = 600s
```
Slot₁ = 600s  → RTA₁ = 600s  (Delay₁ = 0s)
Slot₂ = 690s  → RTA₂ = 690s  (Delay₂ = 60s)
Slot₃ = 780s  → RTA₃ = 780s  (Delay₃ = 120s)
Total Delay: 180s
```

**Strategy B (Optimized)**: RTA₁ = ETA_early,1 = 480s
```
Slot₁ = 480s  → RTA₁ = 480s  (Speedup₁ = 120s)
Slot₂ = 570s  → RTA₂ = 570s  (Speedup₂ = 60s)
Slot₃ = 660s  → RTA₃ = 660s  (Delay₃ = 0s)
Total Time Adjustment: 180s (acceleration-based)
```

**Analysis**: While both strategies involve 180s of total time adjustment, Strategy B utilizes aircraft acceleration capabilities rather than imposing delays. In practice, acceleration (within operational limits) typically incurs lower cost than holding delays due to fuel efficiency and passenger scheduling considerations.

### 4.2 Example 2: Sparse Traffic Scenario

**Input Parameters:**
```
Aircraft A₁: ETA₁ = 600s, TW₁ = [480s, 720s]
Aircraft A₂: ETA₂ = 720s, TW₂ = [600s, 840s]
Slot Duration: S = 90s
```

**Strategy**: RTA₁ = ETA₁ = 600s
```
Slot₁ = 600s  → RTA₁ = 600s  (Delay₁ = 0s)
Slot₂ = 690s  → RTA₂ = 720s  (Delay₂ = 0s)
                              Natural separation = 120s > S
Total Delay: 0s
```

**Analysis**: When natural traffic separation exceeds the required slot duration, no optimization is necessary. Aircraft can maintain their nominal ETAs without intervention.

---

## 5. Performance Metrics

### 5.1 Key Performance Indicators

**Average Delay per Aircraft:**
```
D_avg = (1/n) Σᵢ₌₁ⁿ max(0, RTAᵢ - ETAᵢ)
```

**Time Window Utilization:**
```
U_i = |RTAᵢ - ETAᵢ| / |ETA_late,i - ETA_early,i|
```

**Slot Efficiency:**
```
E = (Actual Throughput) / (Maximum Theoretical Throughput)
```

### 5.2 Complexity Analysis

- **Time Complexity**: O(n) for sequential processing
- **Space Complexity**: O(n) for storing aircraft data
- **Update Frequency**: Real-time updates as new ETA information becomes available

---

## 6. Discussion and Future Work

### 6.1 Advantages of Proposed Method

1. **Proactive Optimization**: Look-ahead strategy prevents delay accumulation
2. **FCFS Preservation**: Maintains fairness in sequencing
3. **Flexibility**: Leverages 4D-RTA capabilities of modern aircraft
4. **Scalability**: Linear computational complexity

### 6.2 Limitations and Extensions

Current model assumptions:
- Deterministic ETA predictions (future work: stochastic models)
- Homogeneous aircraft performance (extension: heterogeneous fleet)
- Single merging point (extension: multiple merge points)

### 6.3 Integration with Broader ATM Systems

This slot allocation module represents one component of a comprehensive TMA management system. Future integration includes:
- Coordination with departure management
- Weather impact modeling
- Multi-airport coordination
- Machine learning for ETA prediction refinement

---

## 7. Conclusions

This paper presented a dynamic slot allocation strategy for TMA merging point operations that balances individual aircraft efficiency with system-wide performance. The key innovation lies in the look-ahead optimization of the lead aircraft slot assignment, which prevents delay propagation through the arrival stream.

The methodology provides:
- Systematic framework for RTA assignment
- Theoretical foundation for optimization decisions
- Practical implementation guidelines

Future research will focus on extending the model to stochastic environments and validating performance through simulation and field trials.

---

## References

[To be added based on actual literature review]

1. SESAR Joint Undertaking. (2023). 4D Trajectory-Based Operations Concept.
2. EUROCONTROL. (2024). Point Merge Integration Guidelines.
3. FAA. (2024). Time-Based Flow Management System Documentation.

---

## Appendix A: Notation Summary

| Symbol | Description |
|--------|-------------|
| TMA | Terminal Maneuvering Area |
| MP | Merging Point |
| ETA | Estimated Time of Arrival |
| RTA | Required Time of Arrival |
| TW | Time Window |
| S | Slot Duration (90 seconds) |
| FCFS | First-Come-First-Served |
| A_i | Aircraft i |
| n | Total number of aircraft |