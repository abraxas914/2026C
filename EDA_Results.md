# 探索性数据分析（EDA）结果：2026 MCM Problem C

数据文件：`2026_MCM_Problem_C_Data.csv`（位于本目录）

## 数据概览

- 数据规模：`421 × 53`
- 字段类型：`float64=44`、`int64=3`、`object=6`
- 覆盖赛季（season）：`1–34`（共 34 季）
- 最终名次（placement）：`1–16`
- 明星年龄（celebrity_age_during_season）：`14–82`，均值 `38.79`
- 重复行（完全相同记录）：`0`

## 缺失值与结构性缺失（最关键）

缺失主要来自两类“结构性原因”，需要在建模/清洗时与“淘汰后记 0 分”严格区分：

1. **评委人数变化（第 4 评委经常不存在）**
   - 缺失率最高的字段几乎都来自 `weekX_judge4_score`（例如 `week11_judge4_score` 缺失率约 `93.8%`）
2. **赛季周数不同（超过赛季长度的周为 N/A）**
   - 例如 `week11_judge1-3_score` 缺失率约 `71.7%`，说明只有少数赛季存在第 11 周

其他较明显缺失：

- `celebrity_homestate` 缺失率约 `13.3%`（通常对应非美国或未记录）

## 类别变量（object）分布

- `celebrity_name`：唯一值 `408`（存在少数重复参赛/回归赛季）
  - 示例（各出现 2 次）：Apolo Anton Ohno、Melissa Rycroft、Bristol Palin、Kelly Monaco、Shawn Johnson
- `ballroom_partner`：唯一值 `60`
  - Top 5（出现次数）：Cheryl Burke `25`、Tony Dovolani `21`、Mark Ballas `21`、Valentin Chmerkovskiy `19`、Karina Smirnoff `18`
- `celebrity_industry`：`26` 类
  - Top 5：Actor/Actress `128`、Athlete `95`、TV Personality `67`、Singer/Rapper `61`、Model `17`
- `celebrity_homestate`：`45` 类（仅对部分记录有效）
  - Top 5：California `72`、New York `47`、Texas `25`、Florida `20`、Illinois `15`
- `celebrity_homecountry/region`：`26` 类
  - Top 5：United States `365`、England `11`、Canada `9`、Australia `4`、Czechoslovakia `3`
- `results`：`17` 类
  - 高频：Eliminated Week 2 `40`、Eliminated Week 7 `36`、Eliminated Week 8 `36`、Eliminated Week 4 `35`
  - `1st/2nd/3rd Place` 各 `34`（与 34 个赛季对应）

## 评委打分字段（weekX_judgeY_score）

- 打分字段数量：`44`
- 非空分数点总数：`13,783`
- 数值分布（跨所有周与评委汇总）：
  - 最小值 `0`，中位数 `7`，均值 `5.235`，最大值 `13.3333`
  - `0` 分数量：`4,671`（与“淘汰后后续周记 0 分”的记录规则一致）
  - `>10` 的分数数量：`221`（可能来自多舞平均、bonus/team 等机制）
  - 负数：`0`

## 赛季结构（人数与周数）

- 每季参赛人数（按 `celebrity_name` 去重计数）：最少 `6`，中位数 `12`，最多 `16`
- 赛季周数（按该季“存在非空评分”的最大周）：分布为
  - `4(1季), 6(1季), 8(1季), 9(2季), 10(20季), 11(9季)`
- 第 4 评委出现频率（各周 `weekX_judge4_score` 的非空记录数）：
  - week1–11：`81/91/128/130/119/147/144/108/133/93/26`

## 一致性检查：results 与评分序列

对 `results` 中的 “Eliminated Week k” 抽取淘汰周，并与“最后一次出现正分的周”比对：

- 共 `298` 条记录带有淘汰周信息
- 两者差值（last_positive_week − eliminated_week）的计数：
  - `0：297` 条（完全一致）
  - `-1：1` 条（建议后续人工核验该条记录/该赛季周次设置）

该检查说明：**用评分序列识别在赛区间/淘汰周是可行的**，但仍需对少数异常做容错。

## 下一步（Next steps）

1. **构建周级“在赛集合”与“淘汰集合”**
   - 将 N/A（缺失）与淘汰后 `0` 分严格区分，避免把缺失当作 0
2. **构造每周总分与标准化分**
   - 定义每周评委总分 `J_{i,t} = sum_j score_{i,t,j}`（跳过缺失评委）
   - 按周做标准化（z-score 或 rank/percent），以提升跨赛季可比性
3. **统计并标注特殊周**
   - 双淘汰/三淘汰周、无淘汰周、评委人数变化周（后续反演 fan votes 需显式处理）
4. **为反演 fan votes 做最小数据集**
   - 生成每季每周：在赛人数、被淘汰者、`J_{i,t}` 列表（用于 rank/percent 规则约束）

