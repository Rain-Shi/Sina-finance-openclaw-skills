# Sina Finance × OpenClaw Skills

基于 [OpenClaw](https://github.com/mariozechner/openclaw) AI Agent 平台 + [新浪财经 MCP](http://mcp.finance.sina.com.cn/) 构建的中文财经助手 Skill 集合。

> **用户用自然语言提问，系统自动调用实时市场数据，输出结构化的财经分析结论——无需记忆股票代码，无需具备专业金融知识。**

---

## 特性

- **自然语言触发**：「茅台多少钱」「今天大盘怎么样」「我持仓今天涨了多少」即可路由到正确的 skill
- **结构化输出**：每个 skill 在 description 中内嵌输出骨架，约束模型按固定模板生成，避免随机发挥
- **实时数据驱动**：所有数值通过 MCP 工具实时获取，禁止使用模型训练知识填充
- **防幻觉硬约束**：研报/财报类 skill 在 description 与 SKILL.md 双层声明「禁止补全原文未披露数据」
- **跨市场覆盖**：A 股 / 港股 / 美股 / 期货 / 外汇 / 文档解析统一入口
- **互斥路由**：边界相近的 skill 在 description 中互写排他声明，避免误触发

## Skills 清单（共 12 个）

### 个股类（4 个）

| Skill | 触发场景 |
|---|---|
| `stock-quick-query` | 「茅台多少钱」「腾讯今天涨了吗」—— 单股现价 / 涨跌速查 |
| `stock-deep-analysis` | 「分析一下宁德时代」「比亚迪能不能买」—— 基本面 / 技术面 / 资金面 / 消息面综合分析 |
| `stock-compare` | 「对比茅台和五粮液」「这两个哪个更值得买」—— 多股横向对标 |
| `portfolio-query` | 「我持有 X、Y、Z，今天整体怎么样」—— 持仓组合加权计算 |

### 板块与市场类（2 个）

| Skill | 触发场景 |
|---|---|
| `sector-hotspot` | 「哪些板块在涨」「白酒板块怎么样」—— 板块热度 + 龙头股 |
| `market-morning-brief` | 「今天大盘」「市场早报」—— 情绪 / 三大指数 / 涨停 / 强势板块 / 要闻一站式快报 |

### 新闻与跨市场类（3 个）

| Skill | 触发场景 |
|---|---|
| `news-digest` | 「今天有什么新闻」—— 政策/监管、宏观、公司/行业、国际市场四分类汇总 |
| `futures-query` | 「黄金多少钱」「原油期货」—— 商品 + 金融期货，含内外盘价差换算 |
| `forex-query` | 「美元兑人民币」「1000 美元能换多少」—— 主要货币对实时汇率 + 换算 |

### 智能研究类（1 个）

| Skill | 触发场景 |
|---|---|
| `smart-research` | 「新能源后面怎么看」「帮我找值得关注的股票」—— 开放性问题意图识别 + 多步研究 |

### 文档解析类（2 个）

| Skill | 触发场景 |
|---|---|
| `research-report-digest` | 上传研报 PDF / 粘贴研报内容 —— 8 段结构化提取，强防幻觉 |
| `financial-report-parse` | 上传财报 PDF / 粘贴财报数据 —— 核心业绩 / 盈利质量 / 分部拆分，页码溯源 |

---

## 安装

### 前置依赖

1. [OpenClaw](https://github.com/mariozechner/openclaw) 已安装并完成 onboard
2. 新浪财经 MCP 已注册到 `~/.openclaw/openclaw.json`
3. 推荐模型：火山引擎豆包 Doubao Seed 2.0 Pro（已实测）/ Claude Sonnet 4 / GPT-5

### 安装步骤

```bash
git clone https://github.com/Rain-Shi/Sina-finance-openclaw-skills.git
cp -r Sina-finance-openclaw-skills/skills/* ~/.openclaw/skills/

# 重启 gateway 让 skills 生效
openclaw gateway restart
# 或手动重启
# pkill -f "openclaw gateway" && openclaw gateway &
```

### 验证

在 OpenClaw 对话界面输入：

```
今天大盘怎么样？
```

若触发 `market-morning-brief` 并按 7 段模板输出，安装成功。

---

## 设计理念

### 1. 骨架塞 description 模式

经实测发现，部分模型（尤其是国产模型如豆包）倾向于**只读 description 字段、跳过 SKILL.md 正文**。本仓库 12 个 skill 中有 11 个把完整输出骨架（① 标题 ② 字段 ③ …）直接写进 description，即使模型不读 SKILL.md，也能按骨架生成。

**例**（节选自 `news-digest`）：

```yaml
description: 用户问"今天有什么新闻""最近市场动态""最新政策"时使用。
  输出必须严格按此结构，禁止自创格式：
  ① 📰今日财经要闻
  ② 🏛️政策/监管
  ③ 📊宏观经济
  ④ 🏢公司/行业
  ⑤ 🌍国际市场
  ⑥ 今日市场关键词
  ⑦ 数据更新时间
  所有新闻必须通过工具实时获取，禁止凭记忆回答。
  完整规则见SKILL.md，必须先用read工具读取后再执行。
```

唯一例外是 `stock-deep-analysis`——它的输出依赖多步工具调用结果，骨架在 SKILL.md 正文中分阶段注入。

### 2. 双层防幻觉（文档解析类）

`research-report-digest` 与 `financial-report-parse` 在 description 与 SKILL.md 中分别声明硬约束：

> **数据错误比没有数据更危险。**
> 原文未披露的字段一律填「原文未披露」，禁止以训练知识补全。

### 3. 双向互斥路由

边界相近的 skill 在 description 中显式互写排他声明，避免误触发：

- `stock-quick-query` ⇔ `portfolio-query`：单股查询 vs 持仓组合
- `stock-deep-analysis` ⇔ `smart-research`：单股深度 vs 开放性研究
- `stock-compare` ⇔ `smart-research`：两股对比 vs 开放性研究

---

## 技术约定

- **语言**：所有输出使用中文，不得出现英文
- **涨跌颜色**：A 股 / 港股 🔴涨 🟢跌（中国习惯）；外汇 🟢涨 🔴跌（国际惯例）
- **单位**：A 股人民币元；港股港元、美股美元（须标注币种）；日元/韩元按 100 单位展示；越南盾/印尼盾按 1000 单位展示
- **数据溯源**：所有数值必须经 MCP 工具实时获取；文档解析类必须标注页码（如 `P.87`）

---

## 评估方法

本仓库配套的 Skill 评估方法学包含 7 个维度：

1. **触发准确性**（F1 ≥ 0.85）
2. **输出质量**（结构合规率 / 字段完整性 / 数据准确性）
3. **增量价值**（with-skill vs baseline 的 Δ Pass Rate）
4. **稳定性**（Pass^5 ≥ 70%）
5. **效率**（Token Efficiency = Δ Pass Rate / Δ Tokens）
6. **安全合规**（负向断言通过率 100%）
7. **工具调用质量**（一票否决）

详细评估框架见配套调研报告（未公开）。

---

## 迭代历程

| 阶段 | 主要工作 |
|---|---|
| v1 | 9 个基础 skill 初稿；规则式 description |
| v2 | 骨架塞 description；双向互斥路由；防幻觉硬约束 |
| v3 | 新增 `research-report-digest` / `financial-report-parse` / `portfolio-query` |
| v4 | `stock-deep-analysis` 升级港股 / 美股路由；`stock-quick-query` × `portfolio-query` 互斥锁 |

---

## 免责声明

本仓库所有 skill 输出**仅供参考，不构成任何投资建议**。投资有风险，决策需谨慎。

## License

MIT

---

*Last updated: 2026-05-12*
