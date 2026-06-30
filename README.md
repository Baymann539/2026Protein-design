# 2026Protein-design
The code for 2026 Protein Design of the Neo-procell team
项目目录/
├── mainproject.py
├── esmscan.py
├── GFP_data.xlsx
├── AAseqs of 5 GFP proteins_20260511.txt
├── Exclusion_List.csv
└── README.md

`mainproject.py`
该脚本负责 GFP 亮度预测与序列生成，主要功能包括：
读取 GFP 亮度训练数据；
提取 avGFP 野生型序列；
根据突变信息生成完整蛋白序列；
使用 ESM 模型生成序列嵌入向量；
使用随机森林模型训练亮度预测模型；
随机生成多突变序列；
预测序列亮度；
排除不允许提交的序列；
输出最终筛选结果。
脚本默认参数包括：
'''python
ESM_MODEL_NAME = "esm2_t12_35M_UR50D"
MAX_MUTATIONS = 6
N_CANDIDATES_TO_GENERATE = 10000
TOP_N_SELECT = 6'''

 `esmscan.py`
该脚本用于对 GFP 序列进行 ESM-Scan 分析，主要功能包括：
自动生成所有单点突变；
或读取用户指定的突变；
使用 ESM 模型计算突变打分；
输出突变打分列表；
生成单点突变热图和箱线图。
ESM-Scan 的结果可作为稳定性筛选参考。一般来说，得分明显较低的突变说明该突变可能破坏蛋白序列的自然性或结构合理性，需要谨慎使用。

输入数据文件
`GFP_data.xlsx`	GFP 亮度训练数据，代码读取其中的 `brightness` 工作表
`AAseqs of 5 GFP proteins_20260511.txt`	存放 GFP 野生型序列，代码从中提取 avGFP 序列
`Exclusion_List.csv`	排除列表，避免输出已经禁止或不应提交的序列

建议使用 Python 3.9 或以上版本。
需要安装的主要依赖如下：
```bash
pip install pandas numpy torch fair-esm scikit-learn biopython tqdm matplotlib openpyxl
```
亮度与稳定性的综合筛选原则
亮度优先  
保留亮度预测值较高的序列，确保改造方向符合提高 GFP 发光效率的目标。
稳定性约束  
使用 ESM-Scan 检查突变位点。如果某个突变在 ESM-Scan 中得分过低，说明该突变可能对蛋白结构不利，应降低优先级或排除。
突变数量控制  
单条序列最多引入 6 个突变，避免突变过多导致结构不可控。
关键区域谨慎处理  
发色团附近位点对亮度影响较大，但也可能影响折叠和成熟过程，因此需要结合亮度预测和 ESM-Scan 结果共同判断。
避免重复和禁止序列  
所有输出序列需要与 `Exclusion_List.csv` 比对，确保不包含不允许提交的序列。

注意事项
ESM-Scan 分数不能直接等同于实验测得的热稳定性，只能作为筛选参考。
随机森林模型的预测效果依赖训练数据质量和数量，需要关注验证集 R²。
随机生成序列可能遗漏更优组合，后续可加入 ProteinMPNN、遗传算法或贝叶斯优化进一步搜索。
最终序列仍需要通过实验检测亮度、成熟效率、表达量和热稳定性。
