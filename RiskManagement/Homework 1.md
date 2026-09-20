##  (一) 设计一个股票的投资组合。

搜集组合中各支股票的历史数据。给出针对该股票组合的一阶近似
$$L_{t+1}^\Delta = -(c_t + b_t' \mathbf{X_{t+1}})$$
对应的 $c_t$ 和 $b_t$，并根据历史数据估计 $\mathbf{X_{t+1}}$ 的联合分布函数。

## 构建方案

### 1. 股票组合的构建

- 对于标的资产与持仓，本次采用了*NASDAQ 100*指数中市值靠前的五只股票（去除了SpaceX是因为历史数据不足），分别为NVDA, AAPL, AMZN, MSFT, GOOGL。为了简便计算与说明，假定每一只股票均持有100股，即持仓向量设为 $\boldsymbol{\lambda} = (100, 100, 100, 100, 100)'$

* 历史数据的选取范围为2023-08-31至2025-08-31，即三年窗口，并选取 $t = \text{2023-08-31}$ 作为分析基准日。

* 基于以上设定，组合基准价值 $V_t$则为各资产市值 $b_{t,i} = \lambda_i S_{t,i}$ ，基准日投资组合总价值 $V_t = \sum_{i=1}^5 \lambda_i S_{t,i} = \$82,772.24$，明细如下：

| 股票代码 (Ticker) | 持股数量 ($\lambda_i$) | 基准日价格 $S_{t,i}$ ($) | 市值 $b_{t,i}$ ($) | 组合权重 $w_{t,i}$ |
| ------------- | ------------------ | ------------------- | ---------------- | -------------- |
| NVDA          | 100                | 49.20               | 4,920.40         | 5.94%          |
| AAPL          | 100                | 185.36              | 18,536.29        | 22.39%         |
| AMZN          | 100                | 138.01              | 13,801.00        | 16.67%         |
| MSFT          | 100                | 320.26              | 32,025.90        | 38.70%         |
| GOOGL         | 100                | 134.89              | 13,488.65        | 16.30%         |
| 合计 ($V_t$)    | -                  | -                   | $82,772.24       | 100.00%        |

&nbsp;

### 2.  组合的一阶线性化损失算子推导

选取对数价格为风险因子 $Z_{t,i} = \ln S_{t,i}$，风险因子的单期增量为**对数收益率** $\mathbf{X}_{t+1} = (X_{t+1,1}, \dots, X_{t+1,5})'$，其中 $X_{t+1,i} = \ln S_{t+1,i} - \ln S_{t,i}$。

组合损失定义为 $L_{t+1}^\Delta = -(V_{t+1} - V_t)$，在单风险因子的设定下，一阶展开形式对应为：
$$L_{t+1}^\Delta = -\left( c_t + \mathbf{b}_t' \mathbf{X}_{t+1} \right)$$
* 常数项 $c_t$：
$$c_t \approx 0.0$$

* 敏感度权重向量 $\mathbf{b}_t$：
$$
\mathbf{b}_{t} = (\lambda_{1} \exp\{Z_{t,1}\}, \lambda_{2} \exp\{Z_{t,2}\}, \dots, \lambda_{5} \exp\{Z_{t,5}\})'  = (\lambda_{1} S_{t,1}, \lambda_{2} S_{t,2}, \dots, \lambda_{5} S_{t,5})'= V_{t} \mathbf{w}_{t}
$$
数值为：
$$\mathbf{b}_t = (4920.40, \; 18536.29, \; 13801.00, \; 32025.90, \; 13488.65)'$$


&nbsp;

### 3. $\mathbf{X}_{t+1}$ 联合分布函数的参数估计

在三年的历史窗口下，历史样本期内有 $n = 750$ 个有效交易日的日对数收益率序列，假定风险因子增量服从**多元正态分布**：

$$\mathbf{X}_{t+1} \sim \mathcal{N}_5(\hat{\boldsymbol{\mu}}, \hat{\boldsymbol{\Sigma}})$$

1. 期望收益率向量 $\hat{\boldsymbol{\mu}}$（样本均值）：

$$\hat{\boldsymbol{\mu}} = \begin{pmatrix} \mu_{\text{NVDA}} \\ \mu_{\text{AAPL}} \\ \mu_{\text{AMZN}} \\ \mu_{\text{MSFT}} \\ \mu_{\text{GOOGL}} \end{pmatrix} = \begin{pmatrix} 0.001980 \\ 0.000727 \\ 0.000877 \\ 0.000630 \\ 0.001257 \end{pmatrix}$$

2. 协方差矩阵 $\hat{\boldsymbol{\Sigma}}$（无偏样本协方差矩阵）：


$$
\begin{aligned}
\hat{\boldsymbol{\Sigma}} &= \frac{1}{n-1}\sum_{s=1}^{n} (\mathbf{X}_{s} - \hat{\boldsymbol{\mu}})(\mathbf{X}_{s} - \hat{\boldsymbol{\mu}})' \\
&=\begin{pmatrix}
0.000871 & 0.000152 & 0.000264 & 0.000207 & 0.000211 \\
0.000152 & 0.000284 & 0.000128 & 0.000096 & 0.000116 \\
0.000264 & 0.000128 & 0.000410 & 0.000176 & 0.000220 \\
0.000207 & 0.000096 & 0.000176 & 0.000268 & 0.000111 \\
0.000211 & 0.000116 & 0.000220 & 0.000111 & 0.000369
\end{pmatrix}

\end{aligned}
$$


&nbsp;

### 4.  组合一阶近似损失分布理论结论

由多元正态分布的线性组合性质，投资组合的单期近似损失 $L_{t+1}^\Delta$ 服从一维正态分布：


$$L_{t+1}^\Delta \sim \mathcal{N}\left( \mu_L, \sigma_L^2 \right)$$

其中：

* 损失均值：$\mu_L = -\mathbf{b}_t' \hat{\boldsymbol{\mu}}$

* 损失方差：$\sigma_L^2 = \mathbf{b}_t' \hat{\boldsymbol{\Sigma}} \mathbf{b}_t$


### 5.  进一步分析

接下来本文计算了每一天资产组合的基于一阶近似结果$L_{t+1} \approx -\left( c_t + \mathbf{b}_t' \mathbf{X}_{t+1} \right)$所算出来的损失，将其与上面得出来的理论分布进行拟合比较，发现拟合结果较好，由此在某种程度上支撑了前面提到的多元正态假设，但依旧存在着一定的肥尾现象，如下图所示：

![[Pasted image 20260911163619.png|659]]


## AI使用声明与代码附录

本次作业借助了AI生成了数据分析的代码和润色了行文，但是资产组合构建思路、股票数据的收集与整理均是独立完成。

```
# %%
# Import packages
import yfinance as yf
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy.optimize import linprog
from scipy import stats
import warnings
warnings.filterwarnings('ignore')

# %%
# select 5 stocks
index_name = "NVDA,AAPL,AMZN,MSFT,GOOGL"
index_stocks = index_name.split(",")
print(index_stocks)

# %%
# download data
stock_tickers = index_name.replace(","," ")
print(stock_tickers)
data = yf.download(stock_tickers, start="2023-8-31", end="2026-8-31", group_by='tickers')
print(data.shape)
display(data.head())

# %%
data_long = (
    data
    .stack('Ticker')  # 把Ticker从列索引压入行，和Date组成双索引
    .reset_index()    # 重置索引，让Date和Ticker成为普通列
    .pivot_table(     # 把Indicator转成列（Open/High/Low等）
        index=['Ticker', 'Date'],
        values=data
        .columns.levels[1],  # 指标列名
        dropna=False
    )
    .reset_index()    # 最终重置索引，得到整洁的长格式
)
# data_long = data_long.drop(columns='Adj Close')
print(data_long.columns)
data_long.columns = ['SecurityID', 'TradingDate', 'ClosePrice', 'HighPrice', 'LowPrice', 'OpenPrice', 'Volume']
data_long = data_long[['SecurityID', 'TradingDate', 'ClosePrice','Volume']]
# 按SecurityID分组，计算每个股票相对上一交易日的收益率
data_long['LnPrice'] = np.log(data_long['ClosePrice'])
data_long['Xit'] = (
    data_long
    .sort_values(['SecurityID', 'TradingDate'])  # 先排序，确保日期顺序正确
    .groupby('SecurityID')['LnPrice']
    .diff()  # 计算相对上一行的变化
)

stocks = data_long['SecurityID'].unique()
display(data_long)

# %%
# -------------------------------------------------------------
# 步骤 1: 确定基准日 t (2023-08-31) 并计算组合价值 V_t
# -------------------------------------------------------------
# 设定每只股票持股数 lambda_i = 100
lambda_shares = pd.Series(100.0, index=index_stocks, name="Shares")

# 提取基准日 t 的收盘价 S_{t, i}
base_date = pd.to_datetime("2023-08-31")
t_prices = (
    data_long[data_long["TradingDate"] == base_date]
    .set_index("SecurityID")["ClosePrice"]
    .reindex(index_stocks)
)

# 计算各资产市值与组合总价值 V_t
asset_values_t = lambda_shares * t_prices
V_t = asset_values_t.sum()

print("=" * 50)
print(f"基准日 t ({base_date.strftime('%Y-%m-%d')}) 投资组合总价值 V_t: ${V_t:,.2f}")
print("=" * 50)

# -------------------------------------------------------------
# 步骤 2 & 3: 计算市值权重 w_{t, i}、向量 b_t 与标量 c_t
# -------------------------------------------------------------
# 计算市值权重 w_{t, i}
w_t = asset_values_t / V_t

# 对应一阶近似系数: L_{t+1}^\Delta = -(c_t + b_t' * X_{t+1})
c_t = 0.0
b_t = asset_values_t.copy()  # b_{t, i} = lambda_i * S_{t, i} = V_t * w_{t, i}

portfolio_summary = pd.DataFrame(
    {
        "Shares": lambda_shares,
        "Price S_t": t_prices,
        "Market Value (b_t)": b_t,
        "Weight (w_t)": w_t,
    }
)
display(portfolio_summary)

print(f"常数项 c_t: {c_t}")
print(f"权重向量 b_t:\n{b_t.to_dict()}")

# -------------------------------------------------------------
# 步骤 4: 估计 X_{t+1} 的联合分布函数 (多元正态分布参数估计)
# -------------------------------------------------------------
# 转换为以日期为索引、各资产收益率为列的宽表格矩阵 (排除缺失的第1天 diff)
X_matrix = (
    data_long.dropna(subset=["Xit"])
    .pivot(index="TradingDate", columns="SecurityID", values="Xit")[index_stocks]
)

# 估计均值向量 mu (d 维) 与 协方差矩阵 Sigma (d x d)
n_obs = len(X_matrix)
mu_hat = X_matrix.mean()
Sigma_hat = X_matrix.cov()  # 默认使用 ddof=1 无偏估计

print("\n" + "=" * 50)
print(f"样本期有效交易日观测数: {n_obs}")
print("\n估计的日期望收益率向量 mu_hat:")
print(mu_hat)
print("\n估计的日收益率协方差矩阵 Sigma_hat:")
display(Sigma_hat)

# -------------------------------------------------------------
# 步骤 5: 导出组合一阶近似损失分布与下一期 (t+1) 实际近似损失
# -------------------------------------------------------------
# 理论分布: L_{t+1}^\Delta ~ N(-b_t' * mu, b_t' * Sigma * b_t)
b_vec = b_t.values
loss_mean = -np.dot(b_vec, mu_hat.values)
loss_variance = np.dot(b_vec, np.dot(Sigma_hat.values, b_vec))
loss_std = np.sqrt(loss_variance)

print("\n" + "=" * 50)
print(f"组合日近似损失的期望值 E[L]: ${loss_mean:.4f}")
print(f"组合日近似损失的标准差 Std[L]: ${loss_std:.4f}")
print(f"理论损失分布: L ~ N({loss_mean:.2f}, {loss_std:.2f}^2)")

# 验证 t+1 日 (下一个交易日 2023-09-01) 的近似损失
trading_dates = sorted(data_long["TradingDate"].unique())
t_plus_1_date = trading_dates[trading_dates.index(base_date) + 1]

X_t_plus_1 = X_matrix.loc[t_plus_1_date]
approx_loss_t_plus_1 = -(c_t + np.dot(b_vec, X_t_plus_1.values))

print(
    f"\n基准日次日 ({pd.to_datetime(t_plus_1_date).strftime('%Y-%m-%d')}) 的一阶近似损失 L_(t+1): ${approx_loss_t_plus_1:.2f}"
)

# %%
# -------------------------------------------------------------
# 1. 计算历史样本序列对应的组合一阶近似损失序列
# -------------------------------------------------------------
# 对历史每一个样本交易日 s，计算近似损失 L_s = -b_t' * X_s
hist_losses = -X_matrix.dot(b_vec)

# -------------------------------------------------------------
# 2. 计算 95% 和 99% 的参数化解析 VaR
# -------------------------------------------------------------
alpha_95, alpha_99 = 0.95, 0.99
# VaR_alpha = mu_L + sigma_L * z_alpha
var_95 = loss_mean + loss_std * stats.norm.ppf(alpha_95)
var_99 = loss_mean + loss_std * stats.norm.ppf(alpha_99)


# -------------------------------------------------------------
# 3. 绘制损失分布密度图与 VaR 标注
# -------------------------------------------------------------
plt.figure(figsize=(11, 6), dpi=300)

# 生成理论正态分布横坐标区间
x_vals = np.linspace(loss_mean - 4 * loss_std, loss_mean + 4 * loss_std, 1000)
norm_pdf = stats.norm.pdf(x_vals, loc=loss_mean, scale=loss_std)

# 绘制历史模拟损失的经验密度直方图与核密度估计 (KDE)
sns.histplot(
    hist_losses,
    stat="density",
    bins=40,
    color="#90CAF9",
    edgecolor="white",
    alpha=0.5,
    label="Historical 1st-Order Loss (Empirical)",
)
sns.kdeplot(
    hist_losses,
    color="#1E88E5",
    linestyle="--",
    linewidth=1.5,
    label="Empirical KDE",
)

# 绘制参数法理论正态密度曲线
plt.plot(
    x_vals,
    norm_pdf,
    color="#E53935",
    linewidth=2.2,
    label=f"Analytical Normal: $\mathcal{{N}}({loss_mean:.1f}, {loss_std:.1f}^2)$",
)

# 填充尾部风险区域 (以 95% VaR 为例) - 修正这里
tail_mask = x_vals >= var_95
plt.fill_between(
    x_vals[tail_mask],
    norm_pdf[tail_mask],
    color="#E53935",
    alpha=0.25,
    label=r"Tail Loss Area ($\alpha \geq 95\%$)",  # 修正：简化 LaTeX
)

# 标注 VaR 阈值虚线
plt.axvline(
    var_95,
    color="#FB8C00",
    linestyle=":",
    linewidth=2,
    label=f"95% VaR: ${var_95:,.2f}",
)
plt.axvline(
    var_99,
    color="#8E24AA",
    linestyle=":",
    linewidth=2,
    label=f"99% VaR: ${var_99:,.2f}",
)

# 格式设置
plt.title(
    f"Portfolio 1st-Order Loss Distribution & Risk Measures (Base Date: {base_date.strftime('%Y-%m-%d')})",
    fontsize=13,
    fontweight="bold",
    pad=12,
)
plt.xlabel(
    r"1-Day First-Order Loss $\tilde{L}_{t+1}$ (\$)", fontsize=11, labelpad=8
)
plt.ylabel("Probability Density", fontsize=11, labelpad=8)
plt.legend( frameon=True, fontsize=10)
plt.grid(True, linestyle="--", alpha=0.4)
plt.tight_layout()

plt.show()



```