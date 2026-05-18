[README.txt](https://github.com/user-attachments/files/27947336/README.txt)
# Bi-Level-EVCS-Data-Center-Power-Grid-Optimization
Bi-Level EVCS + Data Center + Power Grid Optimization
This project implements a complete,modular,and reproducible Python framework for Bi-Level-EVCS-Data-Center-Power-Grid-Co-optimization on the IEEE-118 bus distribution system.
The project replicates baseline methods from the paper (https://doi.org/10.1016/j.apenergy.2025.126971)

🎯Project Objectives

1.Baseline Replication: IT equipment modeling + Cooling system modeling +Thermal coupling considerations + Workload scheduling + Chance-Constrained Programming(CCP) + Bi-level Optimization + Karush–Kuhn–Tucker (KKT) Conditions + Big-M Method + Strong Duality Theorem + Normal Distribution & Inverse CDF + Second-Order Cone Programming (SOCP) & Mixed-Integer Quadratic Programming (MIQP)

2.Combining Reinforcement Learning with bi-level: to handle uncertainty better and make real-time schedule

3.Combining carbon emission cost with objective function

4.Take grid security constraint into consideration

5.Replace chance constraint with deterministic approximation: to faster solving and expand the applicable scenarios from 118-bus to 500-bus

6.Take cooling system and thermal coupling out of consideration: to simplify Data Centre model

7.Turn bi-level into single-level (MPEC relaxation): to simplify the analysis

8.Try to use GNN into optimization

🏗️ Project Structure
01_Code/
├── src/
│   ├── data/                          # 数据加载与预处理模块
│   │   ├── grid_loader.py             # 加载 IEEE-118 和 500-bus (Patch 3) 系统拓扑 [1, 2]
│   │   ├── workload_loader.py         # 处理 IW/BW 计算任务数据 (Alibaba Trace) [3]
│   │   ├── ev_profile_loader.py       # 加载电动汽车充电 sessions (Patch 1) [4]
│   │   └── carbon_data.py             # 获取碳价、配额及排放强度数据 (Patch 2) [3, 5]
│   │
│   ├── models/                        # 物理模型与约束定义
│   │   ├── datacenter/
│   │   │   ├── detailed_dc.py         # Baseline: 含 IT、冷却系统(CRAC/Chiller)及热耦合 [6-10]
│   │   │   └── simplified_dc.py       # Proposed: 剔除冷却系统后的精简数据中心模型 (Goal 6)
│   │   ├── grid/
│   │   │   ├── security_constraints.py# Patch 3: 包含潮流平衡、爬坡限制及热稳定限值 (SCUC) [11-15]
│   │   │   └── objective_builder.py   # Patch 2: 整合发电成本、碳排放交易成本的目标函数 [16, 17]
│   │   └── evcs/
│   │       └── ev_v2g_model.py        # Patch 1: 电动汽车 V2G 交互与旋转备用模型 [18, 19]
│   │
│   ├── math_programming/              # 数学规划与优化重构 (Baseline 核心)
│   │   ├── ccp_linearizer.py          # 机会约束(CCP)的确定性线性化重构 [20, 21]
│   │   ├── kkt_transformer.py         # 利用 KKT 条件将双层模型转化为 MPEC [22, 23]
│   │   └── miqp_solver.py             # 利用 Big-M 和强对偶理论转化为可解的 MIQP [24-26]
│   │
│   ├── learning/                      # AI 驱动的优化与调度 (Patch 1 & 3)
│   │   ├── rl_engine/
│   │   │   ├── environment.py         # 定义基于 MDP 的调度环境 [27]
│   │   │   └── aeppo_agent.py         # Patch 1: 实现 AePPO 算法处理多源不确定性 [28, 29]
│   │   └── gnn_engine/
│   │       ├── graph_builder.py       # 将电网 snapshot 转化为图结构 (Spektral 格式) [30, 31]
│   │       ├── st_gnn_model.py        # Patch 3: 结合 GNN(空间)和 LSTM(时间) 的深度模型 [32, 33]
│   │       └── constraint_reducer.py  # Patch 3: 预测关键线路与机组状态以约简模型 [34, 35]
│   │
│   ├── experiments/                   # 实验运行脚本
│   │   ├── case_baseline_118.py       # 复现原文 Case 1-8 (IEEE 118) [36]
│   │   ├── case_ai_realtime.py        # 运行 RL 辅助的实时调度实验 (Patch 1)
│   │   └── case_large_scale_500.py    # 500 节点大规模系统验证实验 (Patch 3) [37]
│   │
│   └── utils/                         # 性能指标与工具
│       ├── metrics.py                 # 计算 FV (灵活性价值)、CRPS 等评估指标 [38, 39]
│       └── viz_tools.py               # 生成温度曲线、RLMPs 分布及负荷迁移热图 [40, 41]
│
├── configs/                           # 参数配置文件 (.yaml)
├── requirements.txt                   # 依赖包 (Gurobi, PyTorch, Spektral 等)
├── main.py                            # 程序总入口
└── README.md                          # 项目说明文档

🚀 Quick Start(Jupyter Notebook+CPLEX)

1.Clone Repository
git clone https://github.com/anerlin/Bi-Level-EVCS-Data-Center-Power-Grid-Optimization.git
cd Bi-Level-EVCS-Data-Center-Power-Grid-Optimization/01_Code

2.Create Python Environment
conda create -n ecc-opt python=3.10
conda activate ecc-opt

3.Install Python Dependencies
pip install -r requirements.txt

4.Install & Configure CPLEX
①Download and install IBM ILOG CPLEX Optimization Studio:
IBM CPLEX Optimization Studio
②Set environment variable (Linux/Mac example):
export CPLEX_HOME=/path/to/cplex
export PATH=$CPLEX_HOME/bin:$PATH
③Verify Python API:
python -c "import cplex; print(cplex.__version__)"
④Open the Jupyter Notebook
⑤Run Baseline Experiment：
# In notebook cell / 在 notebook cell 中
!python main.py --case baseline_118
⑥Run Proposed Methods：
# RL-assisted real-time scheduling
!python main.py --case rl_realtime
# Carbon-aware dispatch
!python main.py --case carbon_dispatch
# Large-scale 500-bus system
!python main.py --case large_scale_500
⑦Modify Configuration
⑧Verify Environment
⑨Recommended Workflow：
Run baseline IEEE-118 replication→Verify MIQP convergence and RLMP generation→
Enable uncertainty modeling (CCP)→Add carbon-aware objective terms→
Test RL-assisted real-time scheduling→Scale experiments to the 500-bus system

📊 Available Methods

This project integrates a diverse set of methodologies, including Bi-level Optimization, Mathematical Programming with Equilibrium Constraints (MPEC), Chance-Constrained Programming (CCP), and advanced AI techniques like Reinforcement Learning (RL) and Graph Neural Networks (GNN) for spatio-temporal power-compute coordination.

Baseline Methods

The baseline replication follows the methodologies in the original paper:

1.Data Center Modeling: Comprehensive modeling of IT equipment, cooling systems (CRAC/Chiller), and internal thermal coupling effects.

2.Optimization Framework: A Chance-Constrained Bi-level Model coordinating DC operators and power system operators.

3.Uncertainty Handling: Utilizing Chance-Constrained Programming (CCP) based on Normal Distribution and Inverse CDF to manage workload and RES volatility.

4.Mathematical Reformulation: Transforming the bi-level problem into a Single-level MPEC using KKT conditions, and further into an MIQP via the Big-M method and Strong Duality Theorem.

5.Test Case: Validated on the IEEE 118-bus system.

Proposed Methods

Our proposed approach introduces several key improvements and simplifications:

1.Reinforcement Learning (RL) Integration: Combining RL with the bi-level structure to better handle multi-source uncertainties and enable real-time scheduling.

2.Carbon-Aware Objective: Incorporating carbon emission costs directly into the objective function to drive low-carbon dispatch.

3.Security-Constrained Dispatch: Enhancing the power system model with grid security constraints (SCUC-based) to ensure network reliability.

4.Spatio-Temporal Optimization with GNN: Exploring Graph Neural Networks (GNN) to capture network topology features and accelerate optimization.

5.Model Simplification: Removing cooling systems and thermal coupling considerations to streamline the DC model for faster computation.

Optimization & Solving Techniques

1.MPEC Relaxation: Turning the bi-level model into a single-level structure using MPEC relaxation for simplified analysis.

2.Deterministic Approximation: Replacing complex chance constraints with deterministic approximations to enhance solving speed and scalability from 118-bus to 500-bus systems.

3.Hybrid Solver Approach: Utilizing CPLEX for the core MIQP/SOCP tasks while leveraging RL/GNN-assisted modules for decision-making support and constraint reduction.

⚙️ Configuration

The project uses a centralized YAML configuration file (configs/config.yaml) that controls:

1. Model & Physics: 
DC Modeling Mode、Grid Power Flow Formulation、Thermal Safety Boundaries

2. Math Programming & CCP (Uncertainty)：
Chance-Constrained Confidence Level (α)、Reformulation Constants (Big-M)

3. AI-Driven Engines (Learning Engines)：
AePPO Reinforcement Learning Parameters、GNN Reduction Thresholds

4. Economic Settings & Carbon Market：
Carbon Price Setting、Dynamic Pricing Control

🔬 Technical Details

1. Optimization Framework

Architecture: A Chance-Constrained Bi-level Model coordinating Data Center (DC) operators (Upper-level) and Power System operators (Lower-level)

Reformulation: The bi-level structure is transformed into a Single-level Mathematical Program with Equilibrium Constraints (MPEC) via Karush-Kuhn-Tucker (KKT) conditions

Linearization: Big-M Method and Strong Duality Theorem are applied to convert the MPEC into a Mixed-Integer Quadratic Program (MIQP), ensuring global optimality using commercial solvers like CPLEX

Why this approach: Unlike heuristic penalties, this rigorous mathematical reformulation allows for the endogenous formation of Risk-Reflective Locational Marginal Prices (RLMPs), providing precise economic signals for spatial-temporal load migration

2. Advanced Physical Modeling

DC Heterogeneity: Granular modeling of IT equipment (linear power-utilization model) and heterogeneous cooling systems (CRAC vs. Chiller plants)

Thermal Dynamics: Explicit characterization of indoor/outdoor thermal coupling and IT chip temperature evolution using an equivalent R-C thermal inertia model

Grid Security: Implementation of Security-Constrained Unit Commitment (SCUC) with both B-theta and Power Transfer Distribution Factor (PTDF) formulations for large-scale grids (up to 500-bus)

Why this approach: Detailed thermal modeling is essential to capture the DC's true flexibility limits (e.g., using thermal inertia to shift load) without breaching QoS or safety boundaries

3. Uncertainty & Stochasticity 

Methodology: Chance-Constrained Programming (CCP) utilizing the Inverse Cumulative Distribution Function (Inverse CDF) for multi-source uncertainties (Workload, RES, and Temperature)

Assumption: Uncertain variables are modeled as following Normal Distributions, enabling a tractable deterministic linear approximation

Why this approach: CCP offers higher computational efficiency than scenario-based stochastic optimization (SO) for large systems, avoiding the combinatorial explosion of scenario counts in bi-level layers

4. AI-Driven Optimization Engines

Real-time Scheduling: Adaptive Exploration Proximal Policy Optimization (AePPO) algorithm, utilizing LSTM cell states as MDP state inputs to handle dynamic uncertainty

Model Reduction: Spatio-Temporal GNN (ST-GNN) architecture combining Graph Neural Networks (for spatial topology) and LSTM (for temporal coupling) to identify critical lines and generator states

Why this approach: RL enables real-time response to volatility, while GNN-based Reduced-SCUC (R-SCUC) significantly improves computational efficiency in 500-bus systems by disabling non-critical constraints without loss in solution quality

📈 Outputs

Results Files：
1.dispatch_schedules.csv
2.grid_operation_results.csv
3.price_signals_rlmp.json
4.model_reduction_logs.txt

Visualizations:
1.Spatial-Temporal Migration Charts
2.Thermal Dynamics Tracking
3.Probabilistic Forecast Intervals
4.Training & Convergence curves
5.Grid Security Heatmaps

Key Performance Metrics:
Flexibility Value (FV)
Accuracy & Reliability
Efficiency Enhancement

🧪 Experiment Scenarios 

1. Baseline Replication (IEEE 118-bus System)

Replicates the 8 core scenarios from the original paper to validate the electricity-computation co-optimization (ECC) framework.

Case 1 & 2: Deterministic operation (zero uncertainty) comparing Inflexible DC (I-F) vs. Spatial-Temporal Flexibility (ST-F).

Case 3 & 4: Impact of DC-side uncertainties (Workload & Temperature) on I-F vs. ST-F models.

Case 5 & 6: Impact of Grid-side uncertainties (RES & Load) on I-F vs. ST-F models.

Case 7 & 8: Comprehensive coordination under dual-side multi-source uncertainties.

2. Proposed Advanced Methods (Patches 1 & 2)

 Integration of carbon markets and real-time reinforcement learning agents.

Carbon-Aware Dispatch: Evaluating the impact of carbon emission trading costs on DC spatial migration paths.

Real-time RL Response: Using AePPO (Adaptive Exploration PPO) to handle intra-hour volatility that chance constraints cannot fully capture.

EV-V2G Interaction: Analyzing how EV charging stations (EVCS) providing spinning reserves reduces overall system costs.

3. Large-Scale Scalability & Security (Patch 3)

Grid security enforcement and AI-driven model reduction for large networks.

SCUC Security Scenarios: Testing the model under strict transmission thermal limits and ramping constraints.

GNN-Reduced SCUC (R-SCUC): Comparing solving speed and solution quality on the South-Carolina 500-bus system.

Constraint Reduction Analysis: Benchmarking ST-GNN's accuracy in identifying critical vs. non-critical transmission lines.

🔍 Analysis and Interpretation

Key Metrics

1.Flexibility Value (FV): Absolute relative improvement in DC profits and system costs through spatial-temporal load migration.

2.Risk-Reflective Locational Marginal Prices (RLMPs): Endogenously formed price signals reflecting congestion and uncertainty risks.

3.DC Profit & System Cost: Total operational revenue for DC operators and generation/carbon costs for power systems.

4.Computational Speedup: Base-Normalized Time Saved (BNTS) and Base-Normalized Cost (BNC) to validate GNN-based model reduction.

5.IT & Indoor Temperatures: Tracking thermal dynamics of DC server rooms against safety thresholds.

🛠️ Customization

1.Switching Model Fidelity: Modify datacenter/model_type in config.yaml to toggle between the detailed thermal-aware model (including cooling and thermal coupling) and the simplified model

2.Large-Scale Grid Testing:Switch between IEEE 118-bus and SC 500-bus systems by updating the data loader in src/data/grid_loader.py

3.Linearization Tuning:Adjust Big-M constants and CCP confidence levels (ϵ) in configs/config.yaml to observe the sensitivity of profits to operational risks.

📚 Dependencies

Core & Optimization

numpy, pandas, scipy (Data handling)
Mathematical Modeling: Pyomo or Docplex (CPLEX Python API).
Solver: IBM ILOG CPLEX Optimization Studio (v12.10+ recommended).
Data & AI: numpy, pandas, torch, Spektral.

🚨 Troubleshooting 

1.MIQP Infeasibility: Check if Big-M constants are sufficiently large to bound dual variables but not so large that they cause numerical instability. If CPLEX returns "Infeasible," use the conflict refiner tool to identify which KKT or thermal constraints are being violated.

2.GNN Memory Issues: For the 500-bus system, reduce the batch size or use sparse adjacency matrix representations in Spektral.

3.Solver Convergence: If Gurobi fails to converge, adjust the MIPGAP or check the linearized complementarity conditions.

4.RL Training Collapse: If AePPO reward remains stagnant, check the LSTM cell state inputs and PSO adaptive exploration parameters.

5.CPLEX License Error: Ensure the CPLEX_HOME environment variable is set and the license is valid for large-scale 500-bus problems.

📖 References
Baseline Paper 
 Y. Ye, D. Ma, Y. Wu, H. Hu, X. Zhang, C. Liu, and D. Xu, "Harvesting spatial-temporal load migration flexibility of data centers: A chance-constrained bi-level optimization model with endogenously formed risk-reflective locational prices," Applied Energy, vol. 402, p. 126971, 2026.

Patch 1: RL & Uncertainty 
 Y. Li, S. He, Y. Li, L. Ge, S. Lou, and Z. Zeng, "Probabilistic Charging Power Forecast of EVCS: Reinforcement Learning Assisted Deep Learning Approach," IEEE Transactions on Smart Grid (or IEEE Transactions on Industrial Informatics), 2023.

Patch 2: Carbon Market & Bi-level 
 Y. Li, M. Han, Z. Yang, and G. Li, "Coordinating Flexible Demand Response and Renewable Uncertainties for Scheduling of Community Integrated Energy Systems with an Electric Vehicle Charging Station: A Bi-level Approach," IEEE Transactions on Sustainable Energy, 2022.

Patch 3: Grid Security & GNN 
 A. V. Ramesh and X. Li, "Spatio-Temporal Deep Learning-Assisted Reduced Security-Constrained Unit Commitment," IEEE Transactions on Power Systems, 2023.

