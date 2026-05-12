# OpenClaw 财经版 Skills

基于 [OpenClaw](https://github.com/mariozechner/openclaw) 开源 AI Agent 平台构建的中文财经助手 Skills 集合，配合新浪财经 MCP 数据服务使用。

**用户通过自然语言提问，系统自动调用实时市场数据，输出结构化的财经分析结论——无需记忆股票代码，无需具备专业金融知识。**

## ✨ 特性

- **自然语言触发** —— "茅台怎么样？"、"今天大盘好吗？"、"我持有XX、XX，今天盈亏多少？" 即可触发对应 skill
- **结构化输出** —— 每个 skill 定义固定输出骨架，避免大模型随机生成
- **实时数据驱动** —— 所有数值来自实时 MCP 工具调用，禁止凭记忆回答
- **合规底线** —— 涉及投资建议的输出自带风险提示
- **跨市场覆盖** —— A 股、港股、美股、期货、外汇统一入口
- **文档解析** —— 支持上传研报/财报 PDF，带防幻觉保护

## 📦 Skills 清单（共 12 个）

### 🎯 个股类（4 个）

| Skill | 触发场景 |
|---|---|
| **stock-quick-query** | "茅台多少钱"、"XX今天涨了吗" —— 单股现价/涨跌快查 |
| **stock-deep-analysis** | "分析一下XX"、"XX能不能买" —— 基本面/技术面/资金面/消息面六维深度分析 |
| **stock-compare** | "对比XX和XX"、"XX和XX哪个好" —— 多股横向对标 |
| **portfolio-query** | "我持有XX、XX，今天盈亏多少" —— 持仓组合加权计算 |

### 📊 板块与市场类（2 个）

| Skill | 触发场景 |
|---|---|
| **sector-hotspot** | "哪些板块在涨"、"白酒板块怎么样" —— 板块热度 + 龙头股 |
| **market-morning-brief** | "今天大盘"、"市场早报" —— 市场情绪/三大指数/涨停/强势板块/要闻一站式全景 |

### 📰 新闻与跨市场类（3 个）

| Skill | 触发场景 |
|---|---|
| **news-digest** | "今天有什么新闻" —— 宏观/政策/公司/国际四分类汇总 |
| **futures-query** | "黄金多少钱"、"原油期货" —— 商品期货 + 金融期货，含内外盘价差换算 |
| **forex-query** | "美元兑人民币"、"1000美元换多少" —— 主要货币对实时汇率 + 换算 |

### 🔬 智能研究类（1 个）

| Skill | 触发场景 |
|---|---|
| **smart-research** | "新能源后面怎么看"、"帮我找值得关注的股票" —— 开放性问题意图识别 + 多步研究 |

### 📋 文档解析类（2 个）

| Skill | 触发场景 |
|---|---|
| **research-report-digest** | 上传研报 PDF / 粘贴研报文字 —— 8 段结构化提取，带强防幻觉保护 |
| **financial-report-parse** | 上传财报 PDF / 粘贴财报数据 —— 核心业绩/盈利质量/分部拆分，页码溯源 |

## 🚀 安装使用

### 前置依赖

1. [OpenClaw](https://github.com/mariozechner/openclaw) 已安装并完成 onboard
2. [新浪财经 MCP](http://mcp.finance.sina.com.cn/) 已配置在 `~/.openclaw/openclaw.json`
3. 模型推荐：火山引擎豆包 Doubao Seed 2.0 Pro（已测试）/ Claude Sonnet / GPT-5

### 安装方式

```bash
# 克隆到用户级 skills 目录
git clone https://github.com/<your-username>/openclaw-finance-skills.git
cp -r openclaw-finance-skills/skills/* ~/.openclaw/skills/

# 重启 gateway 使 skills 生效
systemctl --user restart openclaw-gateway.service
# 或手动启动
# openclaw gateway &
```

### 验证

在 OpenClaw 对话界面输入：

```
今天大盘怎么样？
```

若触发 `market-morning-brief` 并按模板输出，则安装成功。

## 🧠 设计理念

### 骨架塞 description 模式

经实测发现，部分模型（如豆包）倾向于**跳过 SKILL.md 文件读取**，仅根据 description 字段作答。本仓库大部分 skill 的 description 字段都内嵌输出结构骨架（如 "① 标题 ② 情绪 ③ 三大指数…"），即使模型不读 SKILL.md，也能按骨架生成。

### 双层防幻觉（文档解析类）

研报/财报类 skill 在 description 和 SKILL.md body 里分别设置"严禁补充原文未披露数据"的硬约束：

> **财务数据的错误比没有数据更危险。**

缺字段一律填"原文未披露"，不得以训练知识补全。

### 双向互斥路由

互斥 skill（如 `stock-quick-query` vs `portfolio-query`，`stock-deep-analysis` vs `smart-research`）在各自 description 里显式声明边界，避免误触发。

## 📐 技术约定

- **语言**：所有输出必须使用中文，不得出现英文内容
- **颜色规则**：
  - A 股/港股：🔴涨 🟢跌（中国习惯）
  - 外汇：🟢涨 🔴跌（国际惯例）
- **单位规则**：
  - A 股：人民币元
  - 港股：港元（需标注）
  - 美股：美元（需标注）
  - 日元/韩元：按 100 单位展示
  - 越南盾/印尼盾：按 1000 单位展示
- **数据溯源**：所有数值必须通过 MCP 工具实时获取；文档解析类必须标注页码（如 "P.87"）

## 🛠️ 迭代历程

| 阶段 | 主要工作 |
|---|---|
| v1 | 9 个基础 skill 初稿 |
| v2 | 骨架塞 description、双向互斥路由、防幻觉约束 |
| v3 | 新增 research-report-digest / financial-report-parse / portfolio-query |
| v4 | stock-deep-analysis 升级港股/美股路由、stock-quick-query×portfolio-query 互斥锁 |

## ⚖️ 免责声明

本仓库所有 skill 输出**仅供参考，不构成任何投资建议**。投资有风险，决策需谨慎。

## 📄 License

MIT

---
*Last updated: 2026-05-07*
