SynBio Challenges 2026 GFP Protein Design
本项目为 2026 年 SynBio Challenges GFP 设计赛题的提交方案。以 sfGFP 为骨架，整合文献验证的突变模块、ProteinMPNN 结构生成及二硫键引入策略，通过 ESM-1v 似然值打分与分类排序，最终筛选出 6 条符合要求的设计序列。
一、环境配置
1. 基础运行环境
运行环境：Python 3.11+
推荐硬件：魔搭社区 GPU 实例（V100），镜像选择 PyTorch

3. 依赖安装
bash
pip install -r requirements.txt

核心依赖库：
fair-esm、transformers、xgboost、scikit-learn、pandas、numpy、biopython、torch

4. ESM-1v 模型权重
模型权重约 7 GB，不包含在仓库内。首次运行时会自动从国内镜像站下载，支持断点续传。
手动下载命令：
bash
wget -c https://hf-mirror.com/facebook/esm1v_t33_650M_UR90S_1/resolve/main/esm1v_t33_650M_UR90S_1.pt -P esm_model/

6. 需自行准备的文件
data/Exclusion_List.csv：从比赛官方数据包中复制
esm_model/：模型权重目录，首次运行可自动下载

二、仓库目录结构
text
├── README.md
├── requirements.txt
├── 0.ipynb                          # 完整流程代码
├── fixed_positions.jsonl
├── data/
│   ├── GFP_data.xlsx
│   ├── AAseqs of 5 GFP proteins.txt
│   ├── submission_template.csv
│   └── 2B3P.pdb
├── output/
│   ├── final_6_sequences.csv
│   └── all_candidates_scored_final.csv
├── notebooks/
│   └── 0.ipynb                      # 根目录代码的同步副本
├── ProteinMPNN/
│   ├── protein_mpnn_run.py
│   ├── protein_mpnn_utils.py
│   └── helper_scripts/
└── docs/
    └── design_documentation.pdf     
    
三、设计策略
1. 候选序列生成
共通过三类独立策略生成初始候选池，合并去重后总计 409 条序列：
来源	生成方法	数量
module	在 sfGFP 骨架上排列组合 10 个文献验证突变（每条序列含 4-8 个突变）	247 条
mpnn	ProteinMPNN 从头生成，固定发色团 5Å 内 27 个残基，发色团修复为 SYG	161 条
ssbond	在 sfGFP 骨架上引入 C147-C204 二硫键对（sfroGFP2 文献验证）	1 条

2. 打分与筛选
亮度代理分：ESM-1v 零样本对数似然值，做 0-1 归一化。与实验亮度的 Spearman 秩相关系数为 0.31。
热稳定性代理分：候选序列与 sfGFP 野生型的 ESM-1v 似然值差值，长度归一化后做 0-1 缩放。
综合分：综合分 = 亮度代理分 × 热稳定性代理分
分类排序规则：mpnn 组取前 3，module 组取前 2，ssbond 组取 1，避免 ESM-1v 偏好导致文献验证策略被淘汰。

4. 关于 XGBoost 亮度预测的说明
原计划训练 XGBoost 回归器预测亮度，但五折交叉验证 R² ≈ -0.002。
原因：训练数据（avGFP 单点 / 双点突变）与候选池（ProteinMPNN 从头设计 + sfGFP 多点组合）的特征分布存在系统性差异，模型无法有效泛化。因此最终改用 ESM-1v 似然值作为亮度代理指标。

四、复现步骤
在魔搭社区创建 GPU 实例（V100），将本仓库上传至 /mnt/workspace/gfp_project_final/
将 Exclusion_List.csv 复制到 data/ 目录下
执行 pip install -r requirements.txt 安装依赖
打开 0.ipynb，按顺序执行所有单元格
完整执行流程：
步骤	说明
环境配置	导入依赖库，下载模型权重
读取模板	从 FASTA 文件提取 sfGFP 模板序列
构建突变库	定义 10 个文献验证位点的突变模块库
生成 module 候选	排列组合生成突变序列池
生成 mpnn 候选	ProteinMPNN 设计 + 发色团修复
生成 ssbond 候选	引入 C147-C204 二硫键
合并总池	合并所有候选并去重
硬过滤	格式合规校验 + 排除列表比对
亮度打分	计算 ESM-1v 似然值作为亮度代理分
稳定性打分	计算似然值差值作为热稳定性代理分
综合分计算	亮度分 × 稳定性分得到综合排序分
分类排序	按组别筛选：3 mpnn + 2 module + 1 ssbond
最终审查	校验格式、发色团、二硫键、排除列表
最终输出 6 条序列，保存至 output/final_6_sequences.csv。

五、注意事项
所有序列均通过 Exclusion_List.csv 逐条比对校验，无违规序列
序列长度控制在 220-250 aa，首氨基酸为 M，仅包含 20 种标准氨基酸
所有候选序列均完整保留发色团核心（SYG/TYG），保证荧光功能
ESM-1v 似然值与实验荧光 / 热稳定性的映射关系未经实验验证，仅作为排序指标使用

六、参考文献
Pédelacq J-D, et al. Superfolder GFP. Nat Biotechnol, 2006
Zhang H, et al. mBaoJin. Nat Methods, 2024
Close DW, et al. TGP. Proteins, 2015
Hirano M, et al. StayGold. Nat Biotechnol, 2022

突变位点明细
表格
突变位点	替换氨基酸	功能分类	文献来源
S30R	R	折叠增强	Pédelacq et al., 2006 (Superfolder GFP)
N132D	D	表面电荷优化	QC2-6 (FPbase)
E138D	D	单体化	Hirano et al., 2023 (StayGold)
Y145F	F	疏水核心优化	Pédelacq et al., 2006; MD 模拟验证
S147P	P	热稳定性增强	FPbase 数据库
H148S	S	亮度增强	YuzuFP 2025 (分子动力学指导设计)
P151T	T	结构刚性增强	QC2-6 (FPbase)
L155T	T	结构刚性增强	QC2-6 (FPbase)
I199T	T	量子产率提升	青色 TGP 2025
V206K	K	单体性增强	FPbase 数据库
