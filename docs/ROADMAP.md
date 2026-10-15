# 转型总路线（2026-10-14 定稿）

全职 6-8h/天，动手:学习 ≈ 7:3，主战场 2027 金三银四，2027 年 1 月中下旬开始投递。

| 阶段 | 时间 | 内容 | 交付物 |
|---|---|---|---|
| 0 地基 | 10/14–10/25 | async/API/function calling/structured output/FastAPI + 数据摸底 | 结构化抽取热身工具；naive baseline |
| 1 RAG 深度 | 10/26–11/22 | 解析层（CV 主场）→ 切分 → 混合检索 → rerank → 评测 | **tender-rag 上线**（M1–M5） |
| 2 Agent+MCP | 11/23–12/20 | 手写 agent loop、LangGraph、MCP、Dify 过一遍 | **项目二：SRE 诊断 Agent** |
| 3 推理与工程化 | 12/21–1/3 | vLLM（本地 1.7B + 可选云 4090 压测）、Langfuse、QLoRA 各一次 | 私有化部署版 + 成本/延迟数据 |
| 4 面试冲刺 | 1/5–1/18 | 简历、项目深挖 20 问、八股、系统设计 | 开始投递 |

## GPU 分工（RTX 5060 Laptop 8G）

- **本地跑**：bge-m3 / bge-reranker（各 ~1-2G 显存）、MinerU 文档解析、
  Qwen 1.7B/4B int4（Ollama 试验用）、vLLM serve Qwen 1.7B（学全流程）、QLoRA ≤4B
- **云跑**：生成 API（DeepSeek/Qwen/GLM）；阶段 3 可选租 AutoDL 4090 跑 8B 压测对比（预算 <¥50）
- **注意**：Blackwell 架构（sm_120）需要 CUDA 12.8+ / 新版 vLLM；docker/vLLM 在 WSL2 内跑

## 项目二：两个候选，阶段 2 开始时拍板（二选一）

### 候选 A：SRE 诊断 Agent（运维对象 = 本项目）

不给别人运维，**给 fin-rag 自己运维**——两个项目形成闭环故事。
运维对象：fin-rag 的 Docker Compose 部署栈（fastapi + postgres/pgvector + redis + nginx
+ 一个模拟外部依赖的 flaky 服务）。故障注入脚本造 20 个场景（CPU 打满 / OOM / 磁盘填满 /
依赖崩溃 / 容器暂停 / 慢查询），Agent 经 MCP 只读采集 → LangGraph 诊断循环 →
human-in-the-loop 确认后执行白名单修复命令。评测指标：20 场景定位/修复成功率。
不上 k8s（笔记本跑不动也不必要），面试谈扩展路径即可。
优点：全本地可控、零外部依赖、扣运维部署背景。

### 候选 B：宏观事件历史复盘 Agent（金融方向的正确打开方式）

回答"历史上类似 X 事件发生后，加密货币/美股/黄金/美债的实际表现如何"——
backward-looking，用 AKShare/yfinance 拉真实历史行情，Agent 识别当前事件 →
检索历史类似事件 → 计算事件后 N 天各资产实际收益分布 → 生成带数据的复盘报告。
工具链：新闻/事件检索 + 行情数据 API + 计算沙箱，MCP + LangGraph 编排。
评测：构造 30 个历史事件题，答案用真实行情数据可判分（有 ground truth）。
优点：扣金融叙事、demo 效果炸裂；风险：外部 API 依赖、金融面试需准备
事件研究方法论（混杂因素、样本偏差）的答辩。
红线：只做历史复盘，不做任何前瞻性走势预测。

## Week 1 任务清单（10/14–10/25）

1. [ ] 环境：装 uv、git init + 推 GitHub、Python 3.11+、WSL2 内验证 nvidia-smi
2. [ ] 注册开发者账号并充值/领额度：DeepSeek（¥10）、阿里百炼（Qwen 免费额度）、智谱（GLM flash 免费）
3. [ ] 注册 HuggingFace 并配置国内镜像（HF_ENDPOINT），下载 bge-m3、bge-reranker-v2-m3 验证本地推理
4. [ ] 热身：结构化抽取工具——喂一段年报文本，LLM 输出 JSON（证券代码/公司全称/
       营业收入/归母净利润/审计意见/分红方案），用 function calling + pydantic 校验，
       async + SSE 流式，FastAPI 包一个 /extract 接口
5. [ ] 数据摸底：用 AKShare 从巨潮资讯网批量下载 5-6 家不同行业公司近 3 年年报
       （20+ 份，每份几百页），pdfplumber/PyMuPDF 解析代表性页面（重点：合并
       资产负债表/利润表的跨页表格），把"脏在哪"（表头断裂/跨页/单位混用）写进
       DECISIONS.md
6. [ ] 每天固定 1h 八股：tokenizer / 上下文窗口 / temperature / 幻觉成因（周末我做面试官抽查）
7. [ ] 周日：写周复盘（博客素材）
