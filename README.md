# ClashRules

自用的 Clash / OpenClash 分流规则。基于 [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR) 常用规则，叠加自定义的 **AI 平台**、**金融数据**、**个人代理**与**直连**四层分流，并通过 `url-test` 自动测速组挑选节点。

## 分流架构

```
请求 → 🤖 AI平台 (AI.list)          → AI 专属节点组
     → 💵 金融数据 (02-Finance.list) → 券商/行情节点组
     → 🚀 节点选择 (01-Proxy.list / ProxyLite.list / ProxyMedia) → 常规代理
     → 🌍 全球直连 (00-Direct.list + ACL4SSR 直连规则 + GEOIP,CN)
     → 🐟 漏网之鱼 (FINAL)
```

规则自上而下匹配，命中即停；AI 与金融优先级最高，保证相关流量不受其他规则影响。

## 文件说明

| 文件 | 用途 | 去向 |
|---|---|---|
| `config/final.ini` | OpenClash 自定义规则总入口（ruleset 引用 + 策略组定义） | — |
| `AI.list` | ChatGPT / Claude / Gemini / Copilot 等 AI 平台域名 | 🤖 AI平台 |
| `02-Finance.list` | 老虎 / 长桥 / 富途 / IBKR / 嘉信等券商及 Yahoo Finance | 💵 金融数据 |
| `01-Proxy.list` | 个人常用代理站点（含 TikTok、开发者站点等） | 🚀 节点选择 |
| `ProxyLite.list` | 常见国外域名与服务（ACL4SSR 风格精简版） | 🚀 节点选择 |
| `00-Direct.list` | 直连列表：国内站点、部分券商、香港银行、3DS 验证等 | 🌍 全球直连 |

## 策略组说明

| 策略组 | 类型 | 说明 |
|---|---|---|
| 🚀 节点选择 | select | 手动选节点或交给 ♻️ 自动选择 |
| ♻️ 自动选择 | url-test | **全部节点**参与测速，60s 间隔 / 容差 20ms |
| 💵 金融数据 | select | 可选金融专用组 / 常规代理 / 直连 |
| 💴 金融自动 | url-test | 仅名字含「美国」的节点，测速地址为 Yahoo Finance 行情接口 |
| 🤖 AI平台 | select | 可选 AI 专用组 |
| 🤖 AI自动 | url-test | 仅名字含「美国」的节点，测速地址为 ChatGPT |
| 🌍 全球直连 | select | 默认 DIRECT，可切换为代理兜底 |
| 🐟 漏网之鱼 | select | 未命中任何规则的流量兜底 |

## 使用方法（OpenClash）

1. 服务 → OpenClash → 配置订阅
2. 编辑你的机场订阅，启用**第三方订阅转换**，「远程配置」填入：

   ```
   https://raw.githubusercontent.com/ycy119744220/ClashRules/refs/heads/main/config/final.ini
   ```

   > 也可以直接把 `config/final.ini` 的内容粘贴到 OpenClash 的「自定义规则」中使用。
3. 更新配置即可生效。

> 说明：仓库默认分支为 `main`，所有 `.list` 引用均指向 main 分支；后续只增删 `.list` 内容的话，订阅链接无需改动，OpenClash 重新更新一次配置即可。

## 日常维护

- 新增分流域名 → 编辑对应 `.list`，提交后在 OpenClash 里「更新配置」
- 规则命中优先级 = `final.ini` 中 ruleset 的排列顺序，新增规则文件时注意插入位置
- 部分券商策略：可代理（02-Finance.list）与强制直连（00-Direct.list）按券商分开维护
