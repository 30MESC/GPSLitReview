<details>
<summary><strong>Curriculum Learning in Legged Locomotion: Analysis of Techniques</strong></summary>

# Curriculum Learning in Legged Locomotion: Analysis of Techniques

This report synthesizes curriculum learning techniques identified in the provided literature on learning-based legged locomotion. It covers methods ranging from terrain adaptation to command scheduling and physics relaxation.

## Overview

Curriculum learning in Reinforcement Learning (RL) for robotics involves progressively increasing the difficulty of training tasks to facilitate learning of complex behaviors. The analyzed papers demonstrate a variety of approaches, categorized into:

1.  **Terrain & Environment Curriculum**: Adapting the physical environment (slope, roughness, obstacles).
2.  **Command & Task Curriculum**: Adapting the requested commands (velocity) or task goals (target distance).
3.  **Dynamics Curriculum**: Adapting the physics simulation itself (soft dynamics) to enable exploration.

---

## 1. Adaptive Terrain Curriculum (Lee et al., 2020)

**Paper:** [_Learning Quadrupedal Locomotion over Challenging Terrain_](file:///Users/romesc/WRK/GPSLitReview/Lee-2020-Learning-Quadrupedal-Locomotion-over-Challenging-Terrain.pdf)

### Technique Description

Lee et al. introduce an **adaptive terrain curriculum** using a **Particle Filter** to model the distribution of "desirable" terrain parameters.

- **Mechanism**: A particle filter estimates the distribution of tasks where the agent's failure rate is within a target window (neither too easy nor too hard).
- **Process**: Particles (terrain parameters like step height) are resampled based on the agent's performance, shifting the distribution towards the "frontier of traversability."
- **Goal**: To train a teacher policy that is robust to increasingly difficult unstructured terrain without manual schedule tuning.

---

## 2. Adaptive Velocity Command Curriculum (Margolis et al., 2022)

**Paper:** [_Rapid Locomotion via Reinforcement Learning_](file:///Users/romesc/WRK/GPSLitReview/Margolis-2022-Rapid-Locomotion-via-Reinforcement-Learning.pdf)

### Technique Description

Margolis et al. address high-speed locomotion by learning the **feasible region** of command velocities.

- **Problem**: Randomly sampling high velocities (e.g., max forward speed + max turn) often leads to infeasible commands and instant failure.
- **Mechanism**: **Grid Adaptive Curriculum**. The joint space of commands $(v_x, v_y, \omega_z)$ is discretized into a grid.
- **Update**: Successful traversal in a grid cell allows probability mass to expand to neighboring cells (increasing difficulty).
- **Key Insight**: Explicitly modeling the _joint distribution_ of feasible commands is superior to independent expansion of each dimension.

---

## 3. Top-Specific & Target Curriculum (Hoeller et al., 2023)

**Paper:** [_ANYmal Parkour: Learning Agile Navigation for Quadrupedal Robots_](file:///Users/romesc/WRK/GPSLitReview/Hoeller-2023-ANYmal-Parkour-Learning-Agile-Navigation-for-Quadrupedal-Robots.pdf)

### Technique Description

Hoeller et al. focus on learning agile navigation skills (parkour) using a **hierarchical curriculum**.

- **Skill Curriculum**: Linearly increases obstacle dimensions (e.g., gap width, box height) based on success rates.
- **Target Curriculum**: Used for the navigation policy. Initially places global targets close to the robot and moves them further away as the agent learns to connect skills.
- **Goal**: Ensures the high-level planner learns to chain short-horizon skills into long-horizon paths.

---

## 4. Implicit & Explicit Diversity Curriculum (Heess et al., 2017)

**Paper:** [_Emergence of Locomotion Behaviours in Rich Environments_](file:///Users/romesc/WRK/GPSLitReview/Heess-2017-Emergence-of-Locomotion-Behaviours-in-Rich-Environments.pdf)

### Technique Description

Heess et al. argue that **environmental richness** itself acts as a curriculum ("implicit curriculum").

- **Implicit**: Training on a mixture of terrains (gaps, hurdles, slopes) prevents overfitting to specific patterns and encourages robust, generalized behaviors.
- **Explicit**: They also demonstrate that gradually increasing obstacle dominance (e.g., hurdle height from low to high) significantly improves learning speed compared to training on the hardest setting immediately.
- **Key Insight**: Simple reward functions (forward progress) combined with diverse, curriculum-driven environments lead to the emergence of complex skills (jumping, crouching).

---

## 5. Capability-Based Stepping Stone Curriculum (Xie et al., 2020)

**Paper:** [_ALLSTEPS: Curriculum-driven Learning of Stepping Stone Skills_](file:///Users/romesc/WRK/GPSLitReview/Xie-2020-ALLSTEPS-Curriculum-driven-Learning-of-Stepping-Stone-Skills.pdf)

### Technique Description

Xie et al. propose an **Adaptive Curriculum** based on a "Capability" metric $C_\pi(\psi, \theta)$ for discrete stepping stones.

- **Metric**: $C_\pi$ estimates the expected value/success of the current policy for specific step parameters (yaw $\psi$, pitch $\theta$).
- **Sampling**: The curriculum samples tasks where the capability is intermediate (the "frontier"), avoiding tasks that are too easy (waste of time) or too hard (failure).
- **Comparison**: Shown to be superior to Uniform Sampling and "Difficult-Tasks-Favored" (which focuses heavily on failures).

---

## 6. Game-Inspired Level Curriculum (Rudin et al., 2022)

**Paper:** [_Learning to Walk in Minutes Using Massively Parallel Deep Reinforcement Learning_](file:///Users/romesc/WRK/GPSLitReview/Rudin-2022-Learning-to-Walk-in-Minutes-Using-Massively-Parallel-Deep-Reinforcement-Learning.pdf)

### Technique Description

Rudin et al. leverage massive parallelism (Isaac Gym) with a simple, rule-based **Game-Inspired Curriculum**.

- **Levels**: Each terrain type (stairs, slopes) has discrete difficulty levels.
- **Rules**:
  - **Promotion**: If the robot traverses the terrain (exits the border), it levels up.
  - **Demotion**: If it moves less than half the target distance, it levels down.
  - **Looping**: Agents at the max level loop back to random levels to prevent catastrophic forgetting.
- **Benefit**: Extremely low computational overhead compared to particle filters, ideal for simulating 4000+ robots in parallel.

---

## 7. Soft Dynamics Curriculum (Zhuang et al., 2023)

**Paper:** [_Robot Parkour Learning_](file:///Users/romesc/WRK/GPSLitReview/Zhuang-2023-Robot-Parkour-Learning.pdf)

### Technique Description

Zhuang et al. introduce a **Two-Stage Curriculum** involving **Soft Dynamics Constraints** to solve the exploration problem for high obstacles (parkour).

- **Stage 1 (Soft Dynamics)**: Obstacles are **penetrable**. The agent receives a penalty proportional to the volume/depth of penetration ($r_{penetrate}$).
  - **Curriculum**: Difficulty of obstacles increases as the agent learns to traverse them with minimal penetration.
  - **Insight**: Allows the robot to "pass through" a wall to find the goal state, providing a gradient for learning the jumping motion without getting stuck.
- **Stage 2 (Hard Dynamics)**: Physics are hardened (impenetrable). The policy is fine-tuned from the pre-trained weights.

---

## Comparison Summary

| Paper               | Domain          | Curriculum Target       | Methodology              | Key Feature                                |
| :------------------ | :-------------- | :---------------------- | :----------------------- | :----------------------------------------- |
| **Lee (2020)**      | Rough Terrain   | **Terrain Parameters**  | **Particle Filter**      | Probabilistic "Frontier of Traversability" |
| **Margolis (2022)** | High-Speed Run  | **Command Velocity**    | **Grid Expansion**       | Joint distribution of feasible commands    |
| **Hoeller (2023)**  | Parkour (Nav)   | **Task Param / Target** | **Linear / Distance**    | Hierarchical skill chaining                |
| **Heess (2017)**    | Rich Env        | **Env Diversity**       | **Implicit / Gradual**   | Robustness from richness; emergence        |
| **Xie (2020)**      | Stepping Stones | **Step Parameters**     | **Capability ($C_\pi$)** | Sampling at the edge of capability         |
| **Rudin (2022)**    | Fast Sim        | **Terrain Levels**      | **Game (Rule-based)**    | Promotion/Demotion; Massively Parallel     |
| **Zhuang (2023)**   | Parkour (Agile) | **Physics Dynamics**    | **Soft Constraints**     | Penetrable physics for exploration         |

### Synthesis & Trends

1.  **Automation**: The field has moved from manual schedules (Heess) to automated adaptive systems (Lee, Xie) and back to simple but massive rule-based systems (Rudin) as simulation throughput increased.
2.  **Beyond Terrain**: Curriculum is now applied to **dynamics** (Zhuang) and **commands** (Margolis), not just terrain geometry.
3.  **Exploration vs. Feasibility**:
    - Methods like **Zhuang** (Soft Dynamics) use curriculum to solve the _exploration_ problem (how to find the goal).
    - Methods like **Margolis** (Velocity Grid) and **Lee** (Terrain) use curriculum to discover the _feasibility_ boundary (what is physically possible).

</details>

<details>
<summary><strong>Reward Formulation in Learning-based Legged Locomotion</strong></summary>

# Reward Formulation in Learning-based Legged Locomotion

Reinforcement learning (RL) for legged locomotion relies heavily on the design of the reward function to shape the agent's behavior. While many works aim for a "universal" reward structure, there are distinct variations depending on whether the task is pure locomotion, high-speed running, or agile navigation (parkour).

## 1. Overview of Reward Structure

Most works formulate the reward function $r_t$ as a linear combination of sub-rewards (terms) $r_i$, often including a mix of **positive reinforcements** for task completion and **negative penalties** for regularization (energy, smoothness, safety).

$$ r(s*t, a_t, s*{t+1}) = \sum*i c_i r_i(s_t, a_t, s*{t+1}) $$

Common components include:

- **Command Tracking**: Incentivizing the robot to follow velocity ($v_x, v_y, \omega_z$) or pose commands.
- **Regularization**: Minimizing energy consumption (torques), ensuring smooth control (action rate), and keeping the body stable (upright).
- **Constraints**: Avoiding collisions, joint limits, and stumbling.

## 2. Comparative Analysis of Reward Terms

The following table summarizes the key reward terms used across prominent literature.

| Paper               | Domain          | Primary Task Reward              | Key Regularization / Penalties                             | Specialized Terms                                                                            |
| :------------------ | :-------------- | :------------------------------- | :--------------------------------------------------------- | :------------------------------------------------------------------------------------------- |
| **Heess (2017)**    | Rich Env        | Forward Velocity ($v_x$)         | Uprightness ($n_z$), Torques ($u^2$)                       | Very sparse formulation.                                                                     |
| **Lee (2020)**      | Rough Terrain   | Proj. Velocity ($v_{pr}$)        | Base Motion ($r_b$: roll/pitch/ortho vel), Torques         | **Foot Clearance** ($r_{fc}$): Reward for lifting feet > scan height.                        |
| **Margolis (2022)** | High Speed      | Velocity Tracking (exp)          | Height ($h$), Orientation ($g_{xy}$), Torques              | **Foot Airtime**: Encourages flight phases.                                                  |
| **Rudin (2022)**    | Fast Sim        | Velocity Tracking (exp)          | Vert. Vel ($v_z^2$), Ang. Vel ($\omega_{xy}^2$), Collision | **Action Smoothness**: ($2a_{t-1} - a_{t-2}$).                                               |
| **Xie (2020)**      | Stepping Stones | **Target Reward** (dist to step) | Energy, Posture, Speed Limit                               | **Alive Bonus**: +2 for staying upright.                                                     |
| **Hoeller (2023)**  | Parkour (Nav)   | Pos/Head Tracking                | Accel ($\dot{v}, \dot{\omega}$), Contact Force             | **Stand at Target**: Sparse reward for reaching goals.                                       |
| **Zhuang (2023)**   | Parkour (Agile) | Velocity ($\alpha_1, \alpha_2$)  | Energy, Penetration (Soft Dynamics)                        | **Penetration Depth/Volume**: Penalty for intersecting obstacles during soft-dynamics phase. |

## 3. Detailed Formulation Breakdown

### 3.1. Tracking-Based Locomotion (Lee, Rudin, Margolis)

The standard for blind or perceptive locomotion is to track a commanded velocity vector $(v_x^*, v_y^*, \omega_z^*)$.

- **Rudin et al. (2022)** use a kernel-based error:
  $$ r*{lin} = \exp(-||v*{b,xy}^\ _ - v_{b,xy}||^2 \ \sigma) $$
    They heavily penalize non-commanded motion, particularly vertical velocity ($v_z^2$) and roll/pitch rates ($\omega\*{xy}^2$), to ensure a flat, stable chassis.

- **Lee et al. (2020)** introduce a **Foot Clearance Reward** ($r_{fc}$) specifically for rough terrain. It rewards the agent when the foot in swing phase is higher than the terrain height measured by height-scan sensors ($H_{scan}$):
  $$ r*{fc} = \sum*{i \in I*{swing}} \mathbb{1}(r*{f,i} > \max(H\_{scan,i}) ) $$
  This explicitly encourages the robot to lift its legs over obstacles, rather than just punishing collision.

- **Margolis et al. (2022)** add a **Foot Airtime** reward to encourage flight phases for high-speed running, and penalize base height deviation to maintain a consistent center of mass height.

### 3.2. Target-Based Navigation (Xie, Hoeller)

For tasks like stepping stones or parkour navigation, velocity tracking is insufficient. The agent must reach specific discrete locations.

- **Xie et al. (2020)** use a combination of a sparse **Target Reward** (exponential decay of distance to footstep target) and a dense **Progress Reward** (velocity towards target).

  - _Alive Bonus_: A constant reward (+2) is given for every timestep the robot remains upright, preventing early termination strategies.
  - _Speed Limit_: A penalty is applied if the robot moves unnaturally fast ($||v|| > 1.6 m/s$).

- **Hoeller et al. (2023)** separate rewards into "Locomotion" (tracking) and "Navigation" (reaching global targets). They include a **"Move in direction"** reward ($\cos\langle v_b, r^*-r \rangle$) to guide the robot generally towards the goal even if specific pathfinding is handled by a high-level planner.

### 3.3. Soft Dynamics & Exploration (Zhuang)

**Zhuang et al. (2023)** introduce a novel formulation for learning agile parkour skills (climbing, leaping) by modifying the physics engine itself.

- **Penetration Penalty**: During the "Soft Dynamics" phase, obstacles are permeable. The reward function includes regularizers for **Penetration Depth** and **Penetration Volume**:
  $$ r*{penetrate} = - \alpha*{vol} V*{inter} - \alpha*{depth} d\_{max} $$
  This provides a gradient for the agent to learn to overcome obstacles (by minimizing penetration) rather than receiving a binary failure signal (collision) which provides no gradient.

## 4. Common Regularization Terms (Consensus)

Across all reviewed papers (Ha-2025 survey consistent), the following regularization terms are nearly universal:

1.  **Torque Penalty**: $-||\tau||^2$ (Minimizes energy/heat).
2.  **Joint Velocity Penalty**: $-||\dot{q}||^2$ (Prevents rapid oscillation).
3.  **Action Smoothness**: $-||a_t - a_{t-1}||^2$ or second-order $-||a_t - 2a_{t-1} + a_{t-2}||^2$ (Ensures smooth control inputs).
4.  **Orientation Penalty**: $-||g_{proj} - [0,0,-1]||^2$ (Keeps body upright).
5.  **Collision Penalty**: Counts of collisions with non-foot links (knees, base).

</details>
