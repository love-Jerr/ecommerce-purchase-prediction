# Ecommerce Purchase Prediction
# 电商用户行为数据挖掘与购买预测分析

基于 Kaggle 电商用户行为公开数据集，构建从数据清洗、特征工程到多维可视化及购买行为预测的全链路分析流程。

---

## 项目背景

电商平台每天产生海量点击流数据，但绝大多数用户只浏览、不购买，数据噪声大、难以直接指导业务。

本项目基于公开的电商用户行为数据集，目标是从 6700 万条点击流数据中挖掘用户行为规律，并预测用户在观察期后 **3 天内是否会购买**，为运营决策提供数据支持。

---

## 数据集

- **来源**：[Kaggle - eCommerce behavior data from multi category store](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store)
- **规模**：2019 年 10 月单月，约 6700 万条事件记录
- **字段**：
  - `event_time`：事件时间
  - `event_type`：事件类型（view / cart / purchase）
  - `product_id`：商品 ID
  - `category_id` / `category_code`：品类
  - `brand`：品牌
  - `price`：价格
  - `user_id`：用户 ID
  - `user_session`：会话 ID


---

## 技术栈

| 类别 | 工具 |
| :--- | :--- |
| 数据处理 | Python、Pandas、NumPy |
| 特征工程 | Pandas、DuckDB |
| 建模 | Scikit-learn |
| 可视化 | Matplotlib、Seaborn、Tableau |
| 存储 | Parquet |
| 环境 | Jupyter Notebook |

---
## 项目结构
```
ecommerce-purchase-prediction/
├── README.md
├── requirements.txt
├── .gitignore
├── notebooks/
│   └── ecommerce_purchase_prediction.ipynb
├── data/
│   ├── raw/.gitkeep
│   └── processed/.gitkeep
└── figures/
    └── *.png
```

---

## 主要工作

### 1. 数据清洗与特征工程

- 使用 Pandas **分块读取** 6700 万条点击流数据，避免内存溢出
- 处理缺失值、异常值（负价格、异常时间、非标准事件名）
- 按 30 分钟不活跃切分会话
- 按时间窗口划分**观察期**（构造特征）与**预测期**（生成标签）
- 生成用户级宽表特征：

| 特征类别 | 示例 |
| :--- | :--- |
| 行为次数 | `view_count`、`cart_count`、`purchase_count`、`event_count` |
| 金额特征 | `price_sum`、`price_max`、`avg_price` |
| 活跃度 | `active_days`、`avg_active_hour`、`recency_hours` |
| 转化率 | `cart_rate`、`purchase_rate`、`remove_cart_rate` |

### 2. 多维可视化与报表

- 用户活跃时段热力图（小时 × 事件类型）
- 商品点击 Top 分布
- 类目热度与转化率排名
- 用户转化漏斗（浏览 → 加购 → 购买）

### 3. 购买预测建模

- 构建 **逻辑回归** 与 **随机森林** 二分类模型
- 处理类别不平衡：
  - `class_weight="balanced"`
  - 尝试 SMOTE 过采样
- 使用 Pipeline 封装“缺失值填充 → 标准化 → 模型”
- 评估指标：ROC-AUC、PR-AUC、Recall、F1

---

## 主要结果

### 标签分布

| 标签 | 数量 | 占比 |
| :--- | :--- | :--- |
| 0（不购买） | 2,749,971 | 99.25% |
| 1（购买） | 20,688 | 0.75% |

> 正样本极少，属于**极度不平衡数据**，因此准确率不作为主要评估指标。

### 模型对比

| 模型 | ROC-AUC | PR-AUC | Recall | F1 |
| :--- | :--- | :--- | :--- | :--- |
| 逻辑回归 | 0.778 | 0.061 | 0.640 | 0.044 |
| 随机森林 | 0.780 | 0.058 | 0.590 | 0.047 |

**结论**：

- 两个模型 ROC-AUC 均在 **0.78** 左右，具备一定预测能力
- PR-AUC 相对随机基线（0.0075）提升约 **8 倍**
- 逻辑回归召回率更高（0.64），能抓到更多真实购买用户
- 随机森林误报更少，但漏报更多

### 用户转化漏斗

| 阶段 | 用户数 | 相对浏览转化率 |
| :--- | :--- | :--- |
| 浏览 | 2,984,091 | 100% |
| 加购 | 68,339 | 2.29% |
| 购买 | 23,676 | 0.79% |

### 关键特征（随机森林特征重要性）

- `purchase_rate`（购买率）
- `recency_hours`（最近活跃度）
- `cart_rate`（加购率）
- `purchase_count`（历史购买次数）

> 说明：**加购行为**和**最近活跃度**是预测购买的关键信号。

---

## 业务建议

基于分析结果，提出以下运营策略：

1. **高潜用户召回**：对“最近加购但未购买”的用户发放优惠券，提升转化
2. **活跃时段投放**：根据用户活跃时段热力图，选择高峰时段推送
3. **品类优化**：重点运营高转化率品类，优化低转化率品类的详情页
4. **流失预警**：`recency_hours` 过大的用户，纳入召回名单
