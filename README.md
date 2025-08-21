# 406 Ventures — Monthly AI Paper Roundup 📚🤖
自动抓取近 **30 天** 的前沿 AI 论文、**建模打分**、**生成摘要**，并把“月度 Top 15”榜单自动发布到你的 GitHub 仓库。

> 一句话：抓论文 → 算影响力 → 选 Top → 摘要总结 → 生成 Markdown/CSV → 推到 GitHub。

---

## ✨ 项目亮点

- **多主题覆盖**：默认抓取 *RAG / 强化学习 / 负责任 AI / AI 对齐 / 可解释性 / 对抗样本 / 复合 AI / 本体论 / 神经符号* 等关键词的论文（来自 arXiv）。
- **影响力评分模型**：用历史数据训练 **XGBoost 回归模型**，融合两类信息：
  - **作者影响力**：基于前两位作者 **H-index** 的加权分（通过 `scholarly` 抓取 Google Scholar）。
  - **内容相关性**：以 **TF‑IDF 余弦相似度** + **时间权重（Citation/Days）** 测算和历史高被引论文的相似度。
- **自动摘要**：下载论文 PDF（前若干页），用 OpenAI 模型生成 **面向非技术读者** 的 400 字中文摘要。
- **一键出榜单**：生成 `Top 15` 的 **CSV + README（Markdown）**，并通过 **PyGithub** 推送到你的仓库当中。
- **可扩展/可复用**：主题词、时间窗、模型参数、摘要长度等都能在脚本里快速修改。

---

## 🗂 代码与数据流

```
arXiv (近30天, 多主题)
        │
        ├─> 抓取元数据（Title/Authors/Date/Abstract/PDF）
        │
        ├─> 查询作者 H-index（scholarly/Google Scholar）
        │
        ├─> TF-IDF 内容向量化（与历史数据相似度 + 时间权重）
        │
        ├─> 训练 XGBoost（预测 Citation Count，用于打分排序）
        │
        ├─> 选 Top 15 并下载 PDF → 提取前几页 → OpenAI 摘要
        │
        └─> 导出 CSV & 生成 README（Markdown）→ 用 PyGithub 推到仓库
```

- 主脚本：`406_Ventures_Final_Deliverable.py`
- 关键函数：  
  - `build_model()`：读取历史数据集 → 计算特征 → **GridSearchCV** 选择最优 `XGBRegressor`。  
  - `build_new_papers()`：拉取最近 30 天的 arXiv 论文 + 主题识别 + 作者 H-index 抓取。  
  - `calculate_content_score(...)`：TF‑IDF + 相似度加权 + 时间权重。  
  - `process_pdf(...)`：下载 PDF、抽取文本（PyPDF2）、调用 OpenAI 生成摘要。  
  - `update_dashboard(...)`：用 **PyGithub** 把 CSV/README 推到指定仓库。

---

## 🛠 环境准备

**Python 版本**：建议 3.9–3.11

**安装依赖**
```bash
pip install pandas numpy scikit-learn xgboost shap requests PyPDF2 scholarly openai PyGithub
```
> 备注：`scholarly` 会访问 Google Scholar，可能受网络与风控影响；必要时配置代理或降低请求频率。

---

## 🔐 环境变量与安全

**强烈建议不要在代码里硬编码密钥**。用系统环境变量或 `.env`（并把 `.env` 写入 `.gitignore`）。

```bash
# 必需
export OPENAI_API_KEY="your_openai_api_key"

# 若要自动推送到 GitHub
export GITHUB_TOKEN="your_github_pat_with_repo_scope"
```

> **安全提示**：如果你已经把真实密钥提交过仓库或泄露，请**立即重置/吊销**相应密钥，并清理 Git 历史（如 `git filter-repo`）。

---

## ⚙️ 可配置项（在脚本内修改）

- **主题与检索语句**：在 `build_new_papers()` 中的 `query_topics` 和 `topics_dict`。  
- **时间窗**：默认 `最近 30 天`，可改为 7/60/90 天。  
- **内容打分**：`calculate_content_score()` 中 TF‑IDF 相似度 + `Citation Count` + `time_weights`。  
- **模型搜索空间**：`build_model()` 里的 `param_grid`（`n_estimators / learning_rate / max_depth ...`）。  
- **摘要长度与风格**：`get_prompt()`；默认约 400 字，面向非技术读者。  
- **GitHub 推送目标**：底部常量 `REPO_NAME`，以及 CSV/README 的本地路径。

---

## 🚀 运行步骤（Quickstart）

1) **克隆并进入仓库**
```bash
git clone <your-repo-url>
cd <your-repo>
```

2) **准备环境 & 依赖**
```bash
python -m venv .venv && source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -U pip
pip install pandas numpy scikit-learn xgboost shap requests PyPDF2 scholarly openai PyGithub
```

3) **配置密钥**
```bash
export OPENAI_API_KEY="your_openai_api_key"
export GITHUB_TOKEN="your_github_pat_with_repo_scope"   # 可选，仅在需要自动上传时
```

4) **编辑目标仓库与路径（可选）**  
在脚本底部把：
```python
REPO_NAME = "username/repo"     # 目标 GitHub 仓库
CSV_FILE_PATH = "/abs/path/to/ranked_summaries.csv"  # 你的本地 CSV 路径
```
替换成你自己的值。

5) **启动**
```bash
python 406_Ventures_Final_Deliverable.py
```

6) **产物**
- `top_15_impactful_papers_new.csv`：Top 15 候选清单（含打分）。  
- `ranked_summaries.csv`：Top 15 终版清单 + 摘要。  
- （可选）仓库根目录的 `README.md`：自动生成的月度榜单页面。

---

## 🧩 自定义玩法

- **扩大主题**：把 `query_topics` 中的 `OR` 关键字继续加上新方向（如 *Vision-Language, Multimodal, Agents*）。  
- **替换摘要模型**：把 `process_pdf()` 内的模型名换成你有权限/成本更友好的模型（比如 `gpt-4o` 或轻量模型）。  
- **改进作者影响力**：可叠加机构影响力、合作网络中心性等特征。  
- **更强的 PDF 解析**：当 PDF 结构复杂时，可以尝试 `pymupdf`（fitz）或 `pypdfium2`。  
- **调度自动化**：可用 GitHub Actions/Cron 定时拉取&发布，实现真正的“月更”。

---

## ⚠️ 已知限制

- **Google Scholar 抓取**：`scholarly` 可能被限流或返回不完整数据；请做好错误处理与重试策略。  
- **摘要质量**：只读 PDF 前 *N* 页且模型总结有随机性；重要论文建议人工复核。  
- **评分可解释性**：当前融合了作者与内容特征，但并非学术同行评审；适合“发现候选”，不宜作为唯一判断依据。  
- **开销**：摘要需要调用 API；请监控用量与费用。

---

## 📝 许可证与引用

- 建议以 **MIT** 或 **Apache-2.0** 开源；请在仓库根目录添加 `LICENSE`。  
- 数据与 API：arXiv、Google Scholar（非官方）、OpenAI、GitHub。使用请遵守各自条款。

---

## 🙏 致谢

感谢指导老师与同学提供的建议与数据支持。若你在使用中遇到问题，欢迎提 Issue 交流改进。

