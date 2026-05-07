# Fetch News - 股票新闻获取技能

封装 daily_stock_analysis 项目的新闻搜索架构，用于理解、配置和调试多源新闻抓取流程。

**Repository**: https://github.com/ZhuLinsen/daily_stock_analysis

## 架构概述

```
SearchService (工厂)
  ├── AnspireSearchProvider    (优先)
  ├── BochaSearchProvider
  ├── TavilySearchProvider
  ├── BraveSearchProvider
  ├── SerpAPISearchProvider
  ├── MiniMaxSearchProvider
  └── SearXNGSearchProvider    (兜底)
```

所有 Provider 继承 `BaseSearchProvider`，支持多 Key 轮询 + 熔断。

## 支持的新闻源

| Provider | Env Key | 类型 | 特点 |
|----------|---------|------|------|
| Anspire | `ANSPIRE_API_KEYS` | 商业 API | 中文优化，同 Key 可复用为大模型 |
| Bocha | `BOCHA_API_KEYS` | 商业 API | 中文搜索优化，支持 AI 摘要 |
| Tavily | `TAVILY_API_KEYS` | 商业 API | LLM 场景优化，月 1000 次免费 |
| Brave | `BRAVE_API_KEYS` | 商业 API | 隐私优先，美股资讯补强 |
| SerpAPI | `SERPAPI_API_KEYS` | 商业 API | Google 结果，月 100 次免费 |
| MiniMax | `MINIMAX_API_KEYS` | 商业 API | 结构化搜索结果 |
| SearXNG | `SEARXNG_BASE_URLS` | 自建/公共 | 无配额，私有部署兜底 |

## 配置

```env
# 至少配置一个新闻源
ANSPIRE_API_KEYS=key1,key2          # 推荐，中文优化
SERPAPI_API_KEYS=key1,key2           # 实时金融新闻
TAVILY_API_KEYS=key1                 # 通用新闻
BOCHA_API_KEYS=key1
BRAVE_API_KEYS=key1
MINIMAX_API_KEYS=key1
SEARXNG_BASE_URLS=https://your.instance     # 自建实例
SEARXNG_PUBLIC_INSTANCES_ENABLED=true       # 自动发现公共实例

# 新闻窗口配置
NEWS_MAX_AGE_DAYS=3                  # 新闻最大时效（天）
NEWS_STRATEGY_PROFILE=short          # ultra_short/short/medium/long
```

## 关键文件

| 文件 | 说明 |
|------|------|
| `src/search_service.py` | 所有 Provider + SearchService 实现 |
| `src/config.py` | 搜索相关配置项（660-670 行） |
| `src/core/pipeline.py` | 分析流水线中调用新闻搜索（380-420 行） |
| `src/market_analyzer.py` | 大盘复盘中的新闻搜索（410-440 行） |
| `src/analyzer.py` | LLM Prompt 注入新闻（2084-2098 行） |

## 排查指南

### 新闻为空
1. 确认至少配置了一个新闻源的 API Key
2. 检查 `.env` 中变量名是否正确（注意 `_KEYS` 复数后缀）
3. GitHub Actions 中需在 Secrets 设置对应变量
4. 查看日志：搜索 `search_stock_news` 相关输出
5. SearXNG 默认启用公共实例发现，可作为无 Key 兜底

### Provider 熔断
每个 Key 连续失败 3 次后自动跳过，冷却后恢复。查看日志中的 `circuit_breaker` 相关输出。

### 新闻质量
- 中文股票推荐 Anspire / Bocha / Tavily
- 美股推荐 Brave / SerpAPI
- 自建 SearXNG 可完全控制搜索结果
