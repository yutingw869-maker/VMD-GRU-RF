# Financial Volatility Proxy Forecasting: VMD-GRU-RF & CEEMDAN-LSTM Baseline

本项目实现了用于金融时间序列波动率代理指标预测的混合建模流水线，包含主模型 **VMD-GRU-RF（变分模态分解 + 分量级 GRU + 随机森林非线性重构）** 与基线对比模型 **CEEMDAN-LSTM（自适应噪声完备经验模态分解 + 分量级 LSTM + 直接求和重构）**。

---

## 1. 项目概述与方法架构

金融市场波动率序列通常呈现非线性、高噪声、异方差及多尺度波动特征。为降低单一时序模型同时学习多频波动模式的难度，本项目采用 **“序列分解 — 分量预测 — 结果重构”** 范式：

1. **波动率代理指标构建**：基于收盘价序列计算对数收益率 $r_t = \ln(P_t / P_{t-1})$，并构建日度波动率代理指标 $RV_t = \vert{}r_t\vert{}$（支持日频数据直接读取，也兼容高频数据按日聚合）。
2. **信号分解**：
   * **VMD-GRU-RF**：使用变分模态分解（VMD，$K=5, \alpha=2000$）将波动率代理序列分解为 5 个分量（`IMF1`–`IMF4` 与 `Res`）。
   * **CEEMDAN-LSTM Baseline**：使用 CEEMDAN 对序列进行模态分解并提取 IMF 与残差项。
3. **分量时序预测**：按 `7:2:1` 时间顺序划分训练、验证与测试集，采用滑动窗口（`window_size = 26`）构建监督样本，分别为各分量训练循环神经网络（GRU / LSTM）预测未来时序走势。
4. **预测重构与评估**：
   * **VMD-GRU-RF**：将各分量预测值拼接为特征向量，使用随机森林回归器（`RandomForestRegressor`）学习非线性重构映射。
   * **CEEMDAN-LSTM Baseline**：将各分量预测结果直接求和得到最终预测值。

---

## 2. 仓库文件说明

```text
.
├── VMD_GRU_RF.ipynb                     # 主模型：VMD + 分量 GRU + RF 非线性重构
├── CEEMDAN_LSTM_BASELINE.ipynb          # 基线模型：CEEMDAN + 分量 LSTM + 直接求和
├── P888.csv                             # 聚丙烯期货主力合约数据（如未上传需自行放置）
├── CSI300.csv                           # 沪深300股指期货主力合约数据（如未上传需自行放置）
└── README.md                            # 项目说明文档
