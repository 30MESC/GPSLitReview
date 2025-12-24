<details>
<summary><strong>Reward Formulation in Learning-based Legged Locomotion</strong></summary>

# Reward Formulation in Learning-based Legged Locomotion

Reinforcement learning (RL) for legged locomotion relies heavily on the design of the reward function to shape the agent's behavior. While many works aim for a "universal" reward structure, there are distinct variations depending on whether the task is pure locomotion, high-speed running, or agile navigation (parkour).

## 1. Overview of Reward Structure

Most works formulate the reward function $r_t$ as a linear combination of sub-rewards (terms) $r_i$, often including a mix of **positive reinforcements** for task completion and **negative penalties** for regularization (energy, smoothness, safety).

70056 r(s_t, a_t, s_{t+1}) = \sum_i c_i r_i(s_t, a_t, s_{t+1}) 70056

Common components include:

- **Command Tracking**: Incentivizing the robot to follow velocity ($v_x, v_y, \omega_z$) or pose commands.
- **Regularization**: Minimizing energy consumption (torques), ensuring smooth control (action rate), and keeping the body stable (upright).
- **Constraints**: Avoiding collisions, joint limits, and stumbling.

## 2. Comparative Analysis of Reward Terms

The following table summarizes the key reward terms used across prominent literature.

| Paper | Domain | Primary Task Reward | Key Regularization / Penalties | Specialized Terms |
| :--- | :--- | :--- | :--- | :--- |
| **Heess (2017)** | Rich Env | Forward Velocity ($v_x$) | Uprightness ($n_z$), Torques ($u^2$) | Very sparse formulation. |
| **Lee (2020)** | Rough Terrain | Proj. Velocity ($v_{pr}$) | Base Motion ($r_b$: roll/pitch/ortho vel), Torques | **Foot Clearance** ($r_{fc}$): Reward for lifting feet > scan height. |
| **Margolis (2022)** | High Speed | Velocity Tracking (exp) | Height ($h$), Orientation ($g_{xy}$), Torques | **Foot Airtime**: Encourages flight phases. |
| **Rudin (2022)** | Fast Sim | Velocity Tracking (exp) | Vert. Vel ($v_z^2$), Ang. Vel ($\omega_{xy}^2$), Collision | **Action Smoothness**: ($2a_{t-1} - a_{t-2}$). |
| **Xie (2020)** | Stepping Stones | **Target Reward** (dist to step) | Energy, Posture, Speed Limit | **Alive Bonus**: +2 for staying upright. |
| **Hoeller (2023)** | Parkour (Nav) | Pos/Head Tracking | Accel ($\dot{v}, \dot{\omega}$), Contact Force | **Stand at Target**: Sparse reward for reaching goals. |
| **Zhuang (2023)** | Parkour (Agile) | Velocity ($\alpha_1, \alpha_2$) | Energy, Penetration (Soft Dynamics) | **Penetration Depth/Volume**: Penalty for intersecting obstacles during soft-dynamics phase. |

## 3. Detailed Formulation Breakdown

### 3.1. Tracking-Based Locomotion (Lee, Rudin, Margolis)

The standard for blind or perceptive locomotion is to track a commanded velocity vector $(v_x^*, v_y^*, \omega_z^*)$.

- **Rudin et al. (2022)** use a kernel-based error:
  70056 r_{lin} = \exp(-||v_{b,xy}^* - v_{b,xy}||^2 / \sigma) 70056
    They heavily penalize non-commanded motion, particularly vertical velocity ($v_z^2$) and roll/pitch rates ($\omega_{xy}^2$), to ensure a flat, stable chassis.

- **Lee et al. (2020)** introduce a **Foot Clearance Reward** ($r_{fc}$) specifically for rough terrain. It rewards the agent when the foot in swing phase is higher than the terrain height measured by height-scan sensors ($H_{scan}$):
  70056 r_{fc} = \sum_{i \in I_{swing}} \mathbb{1}(r_{f,i} > \max(H_{scan,i}) ) 70056
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
  70056 r_{penetrate} = - \alpha_{vol} V_{inter} - \alpha_{depth} d_{max} 70056
  This provides a gradient for the agent to learn to overcome obstacles (by minimizing penetration) rather than receiving a binary failure signal (collision) which provides no gradient.

## 4. Common Regularization Terms (Consensus)

Across all reviewed papers (Ha-2025 survey consistent), the following regularization terms are nearly universal:

1.  **Torque Penalty**: $-||\tau||^2$ (Minimizes energy/heat).
2.  **Joint Velocity Penalty**: $-||\dot{q}||^2$ (Prevents rapid oscillation).
3.  **Action Smoothness**: $-||a_t - a_{t-1}||^2$ or second-order $-||a_t - 2a_{t-1} + a_{t-2}||^2$ (Ensures smooth control inputs).
4.  **Orientation Penalty**: $-||g_{proj} - [0,0,-1]||^2$ (Keeps body upright).
5.  **Collision Penalty**: Counts of collisions with non-foot links (knees, base).

</details>
