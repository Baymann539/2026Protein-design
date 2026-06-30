# 2026 Protein Design

> Code for the 2026 Protein Design project of the Neo-procell team.

本项目面向 **GFP 蛋白序列设计**，主要用于 GFP 亮度预测、候选突变序列生成，以及基于 ESM-Scan 的突变合理性筛选。整体流程结合了 ESM 蛋白语言模型嵌入、随机森林回归模型和突变位点扫描结果，用于辅助筛选更有潜力的 GFP 改造序列。

---

## 目录

- [项目结构](#项目结构)
- [文件说明](#文件说明)
- [环境依赖](#环境依赖)
- [快速开始](#快速开始)
- [主流程说明](#主流程说明)
- [默认参数](#默认参数)
- [筛选原则](#筛选原则)
- [注意事项](#注意事项)
- [后续改进方向](#后续改进方向)

---

## 项目结构

```text
2026Protein-design/
├── mainproject.py
├── esmscan.py
├── GFP_data.xlsx
├── AAseqs of 5 GFP proteins_20260511.txt
└── README.md
```

> 说明：`mainproject.py` 中会尝试读取 `Exclusion_List.csv` 作为排除列表。如果需要使用排除列表，请将该文件放在项目根目录，并确保其中包含 `sequences-not-submit` 列。

---

## 文件说明

| 文件 | 作用 |
| --- | --- |
| `mainproject.py` | GFP 亮度预测与候选序列生成主脚本。 |
| `esmscan.py` | GFP 突变位点的 ESM-Scan 分析脚本。 |
| `GFP_data.xlsx` | GFP 亮度训练数据，代码默认读取其中的 `brightness` 工作表。 |
| `AAseqs of 5 GFP proteins_20260511.txt` | GFP 野生型序列文件，代码会从中提取 avGFP 序列。 |
| `Exclusion_List.csv` | 可选排除列表，用于避免输出禁止提交或不应提交的序列。 |
| `README.md` | 项目说明文档。 |

---

## 环境依赖

建议使用 **Python 3.9 或以上版本**。

安装主要依赖：

```bash
pip install pandas numpy torch fair-esm scikit-learn biopython tqdm matplotlib openpyxl
```

如果希望使用虚拟环境，可以参考：

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate

pip install pandas numpy torch fair-esm scikit-learn biopython tqdm matplotlib openpyxl
```

---

## 快速开始

### 1. 运行 GFP 亮度预测与候选序列生成

```bash
python mainproject.py
```

`mainproject.py` 的主要功能包括：

1. 读取 GFP 亮度训练数据；
2. 提取 avGFP 野生型序列；
3. 根据突变信息生成完整蛋白序列；
4. 使用 ESM 模型生成序列嵌入向量；
5. 使用随机森林模型训练亮度预测模型；
6. 随机生成多突变候选序列；
7. 预测候选序列亮度；
8. 排除不允许提交的序列；
9. 输出最终筛选结果。

### 2. 运行 ESM-Scan 突变分析

```bash
python esmscan.py
```

`esmscan.py` 的主要功能包括：

1. 自动生成所有单点突变，或读取用户指定的突变；
2. 使用 ESM 模型计算突变打分；
3. 输出突变打分列表；
4. 生成单点突变热图和箱线图。

ESM-Scan 的结果可作为稳定性和序列自然性筛选参考。一般来说，得分明显较低的突变可能会破坏蛋白序列的自然性或结构合理性，需要谨慎使用。

---

## 主流程说明

项目整体思路如下：

```text
输入数据
  ├── GFP 亮度数据
  ├── avGFP 野生型序列
  └── 排除列表（可选）
        ↓
序列预处理
  ├── 解析突变字符串
  ├── 生成完整突变序列
  └── 清理无效样本
        ↓
特征提取
  └── 使用 ESM 模型生成蛋白序列嵌入
        ↓
亮度建模
  └── 使用随机森林模型预测 GFP 亮度
        ↓
候选生成
  ├── 随机生成多突变序列
  ├── 预测候选序列亮度
  └── 过滤重复或禁止提交的序列
        ↓
稳定性参考
  └── 结合 ESM-Scan 结果评估突变合理性
        ↓
最终候选序列
```

---

## 默认参数

`mainproject.py` 中的默认参数如下：

```python
ESM_MODEL_NAME = "esm2_t12_35M_UR50D"
MAX_MUTATIONS = 6
N_CANDIDATES_TO_GENERATE = 10000
TOP_N_SELECT = 6
```

| 参数 | 含义 |
| --- | --- |
| `ESM_MODEL_NAME` | 用于生成蛋白序列嵌入的 ESM 模型名称。 |
| `MAX_MUTATIONS` | 单条候选序列最多允许引入的突变数量。 |
| `N_CANDIDATES_TO_GENERATE` | 随机生成的候选序列数量。 |
| `TOP_N_SELECT` | 最终保留的候选序列数量。 |

---

## 筛选原则

### 1. 亮度优先

优先保留预测亮度较高的序列，确保改造方向符合提高 GFP 发光效率的目标。

### 2. 稳定性约束

使用 ESM-Scan 检查突变位点。如果某个突变在 ESM-Scan 中得分过低，说明该突变可能对蛋白结构不利，应降低优先级或排除。

### 3. 控制突变数量

单条序列最多引入 6 个突变，避免突变过多导致结构不可控。

### 4. 关键区域谨慎处理

发色团附近位点对亮度影响较大，但也可能影响折叠和成熟过程，因此需要结合亮度预测和 ESM-Scan 结果共同判断。

### 5. 避免重复和禁止序列

所有输出序列需要与 `Exclusion_List.csv` 进行比对，确保不包含不允许提交的序列。

---

## 注意事项

1. ESM-Scan 分数不能直接等同于实验测得的热稳定性，只能作为筛选参考。
2. 随机森林模型的预测效果依赖训练数据质量和数量，需要关注验证集的 R² 指标。
3. 随机生成序列可能遗漏更优组合，后续可以加入更系统的搜索策略。
4. 最终候选序列仍需要通过实验检测亮度、成熟效率、表达量和热稳定性。

---

## 后续改进方向

后续可以考虑加入以下优化：

- 使用 ProteinMPNN 进行更有结构约束的序列设计；
- 使用遗传算法或贝叶斯优化替代纯随机搜索；
- 将输出文件统一保存到 `outputs/` 目录；
- 新增 `requirements.txt`，方便一键安装依赖；
- 为 `mainproject.py` 和 `esmscan.py` 增加命令行参数；
- 补充示例输入、示例输出和运行截图，方便复现实验流程。

