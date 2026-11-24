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

## 3. 数据说明与准备

模型默认在主程序入口读取以下两个数据集（若文件不存在会自动跳过并提示）：
* `P888.csv`：商品期货主力合约数据（日度有效样本 2,917 条）[cite: 10, 17, 18]
* `CSI300.csv`：沪深 300 股指期货主力合约数据（日度有效样本 2,677 条）[cite: 10, 17, 18]

> **数据格式说明**：脚本会自动识别 `trading_date` 或 `date` 日期列以及 `close` 收盘价列（若含高频日内记录，将按日期聚合日收盘价并计算波动率代理指标）[cite: 7, 17, 18]。请确保本地数据路径与 Notebook 中 `datasets` 配置一致。

---

## 4. 环境依赖与运行方式

### 环境配置
建议在 Python 3.10 环境下安装所需依赖：

```bash
pip install numpy pandas matplotlib scipy statsmodels scikit-learn tensorflow vmdpy EMD-signal

## 5.运行步骤
将 P888.csv 与 CSI300.csv 放置于工作目录（或修改代码中对应的文件路径）。
打开并顺序运行 VMD_GRU_RF.ipynb 或 CEEMDAN_LSTM_BASELINE.ipynb[cite: 17, 18]。
运行完成后，脚本会自动生成：可视化图表：原始波动率序列图（Figure 1）、分解分量图（Figure 2）、各分量预测对比图（Figure 3）、最终预测对比图（Figure 4）[cite: 17, 18]。统计与评估结果：描述统计表、模型评估指标 CSV 文件，以及可直接用于论文排版的 LaTeX 三线表输出[cite: 17, 18]。
