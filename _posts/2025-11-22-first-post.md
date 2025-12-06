---
title: Deep Reinforcement Learning from Human Preferences
date: 2025-11-22 18:00:00 +0800
categories: [强化学习, 文献阅读笔记]

---

## 1. 基本信息
- **论文标题**：Deep Reinforcement Learning from Human Preferences
- **中文标题**：基于人类偏好的深度强化学习
- **作者**：Paul F Christiano, Jan Leike, Tom B Brown, Miljan Martic, Shane Legg, Dario Amodei
- **发表会议**：NIPS 2017
  
---

## 2. 研究背景
- **核心难题**：在许多复杂的强化学习（RL）任务中，设计一个能够准确反映人类意图的**奖励函数（Reward Function）**极其困难，甚至是不可能的。
- **对齐风险**：如果仅使用简单的代理奖励（proxy reward），智能体可能会为了最大化分数而产生不符合人类期望的行为，导致**目标不匹配（Objective Mismatch）** 。
- **现有瓶颈**：
    - **模仿学习**：难以应用于人类无法直接演示的行为（如控制多自由度的异形机器人）。
    - **直接反馈**：直接将人类反馈作为奖励信号成本过高，因为现代深度 RL 系统需要数千小时的经验。
- **本文目标**：通过极少量的人类反馈（不到总交互量的 1%），在没有预定义奖励函数的情况下解决复杂的 RL 任务。

---

## 3. 核心方法
本文提出了一种**从人类对轨迹片段的比较中学习奖励函数**的方法。系统由三个异步运行的进程组成：

### 3.1 进程一：策略优化 (The Policy)
- 智能体与环境交互，生成轨迹 $\{\tau^1, ..., \tau^i\}$ 。
- 使用传统的强化学习算法（Atari 使用 A2C，机器人任务使用 TRPO）更新策略 $\pi$ 。
- **关键点**：策略优化的目标是最大化由**奖励预测网络** $\hat{r}$ 给出的预测奖励总和，而非环境奖励。

### 3.2 进程二：偏好收集 (Preference Elicitation)
- 系统从生成的轨迹中截取短片段（1-2秒），成对展示给人类 $(\sigma^1, \sigma^2)$。
- 人类评估员进行三选一判断：
    1. 片段 1 更好
    2. 片段 2 更好
    3. 同等/无法比较
- 这些数据被存入数据库 $\mathcal{D}$。

### 3.3 进程三：奖励函数拟合 (Reward Function Fitting)
- 使用深度神经网络拟合奖励函数 $\hat{r}$。
- **数学原理**：将奖励视为潜变量，假设人类选择片段 $\sigma^1$ 优于 $\sigma^2$ 的概率服从 **Bradley-Terry 模型**：

$$
\hat{P}[\sigma^{1}>\sigma^{2}] = \frac{\exp\sum \hat{r}(o_{t}^{1},a_{t}^{1})}{\exp\sum \hat{r}(o_{t}^{1},a_{t}^{1}) + \exp\sum \hat{r}(o_{t}^{2},a_{t}^{2})}
$$

- **损失函数**：通过最小化预测概率与人类实际选择之间的**交叉熵损失**来训练网络。

---
