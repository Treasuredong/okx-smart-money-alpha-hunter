---
name: okx-smart-money-alpha-hunter
description: 用于构建基于 OKX Onchain OS 的 Agentic Wallet 自动交易策略，聚焦“聪明钱/KOL/巨鲸信号扫描 + 链上现货自动执行 + 结构化选币 + 严格风控 + 配套可视化面板”的完整工作流。适用于希望在 Solana 与 X Layer 上寻找高质量短中周期机会，并用 Ethereum / BNB Chain 做侦察参考的自动化策略场景。关键词覆盖：聪明钱策略、自动交易、KOL 信号、巨鲸信号、Alpha Hunter、Solana、X Layer、自动买卖、链上现货、风控、选币、动量、流动性、叙事轮动、信号衰减。
license: MIT
metadata:
  author: OpenAI + OKX_DEX
  version: "2.2.0-execution"
  primaryChains:
    - xlayer
    - solana
  scoutChains:
    - ethereum
    - bsc
  dependencies:
    - okx-agentic-wallet
    - okx-dex-market
    - okx-dex-signal
    - okx-dex-token
    - okx-security
    - okx-dex-swap
---

# OKX Smart Money Alpha Hunter: Execution Edition

这是一个专门为 `Agentic Wallet` 设计的自动交易 Skill。

它不讨论“目标口号”，只关注三件事：

1. **更早发现有持续性的买盘**
2. **只在退出也有把握的地方下单**
3. **尽量把浮盈变成已实现利润**

## 一句话定位

这是一个“**聪明钱共振 + 巨鲸确认 + 叙事中继 + 出口优先 + 小资金硬风控**”的 OKX Onchain OS 现货策略。

核心由三条结构组成：

- `Impulse Lane`
  偏向更强动量、更快爆发的轻仓单，适合捕捉短时间加速
- `Flow Lane`
  偏向更深流动性、更稳结构、退出更容易的趋势单
- `Whale Relay Trigger`
  当热点已形成且巨鲸/聪明钱包同步追入时，允许做受控追击，而不是机械错过
- `Reserve Buffer`
  为 gas、滑点和连续决策留出缓冲，不把整只钱包一次性压满

## 必须使用的技能顺序

1. `okx-agentic-wallet`
2. `okx-dex-market`
3. `okx-dex-signal`
4. `okx-dex-token`
5. `okx-security`
6. `okx-dex-swap`

不要把 Web 搜索、第三方价格站、非 onchainOS 交易工具作为主信息源。
**Onchain OS 必须是主要信息源和主要交易工具。**

## 触发规则

以下表达都应该触发本 Skill：

- “帮我做一个聪明钱自动交易策略”
- “帮我做一个 Solana + X Layer 的自动交易 agent”
- “自动扫 KOL / 巨鲸信号并自主买卖”
- “给我一个可以自己找标的并下单的 skill”
- “帮我做一个专注赚钱的链上现货策略”
- “根据钱包余额自动控制仓位的聪明钱策略”
- “把 Ethereum / BNB 只当侦察链的策略写出来”
- “盯紧巨鲸信号，热点起来时敢追但别盲追”
- “做一个更像真人交易员的聪明钱追击策略”

以下表达不应误触：

- 单纯查钱包余额
- 单纯手动 swap
- 单纯导出日志
- 单纯查看某个外部活动状态

这些应该分别交给：

- `okx-agentic-wallet`
- `okx-dex-swap`
- `okx-audit-log`
- 对应的外部状态类 Skill

## 主战场与侦察场

### 主战链

只允许在这两条链自动开仓和自动平仓：

- `xlayer`
- `solana`

### 侦察链

可以读取信号做横向参考，但**不自动交易**：

- `ethereum`
- `bsc`

原因：

- 主战链上更适合小资金快速执行
- 侦察链能更早暴露叙事轮动和聪明钱迁移
- 把“发现”和“执行”分层，有利于减少无效交易

## 状态文件

策略运行时必须加载或初始化：

- `.codex/state/smart-money-alpha-hunter.json`

至少保存这些顶层字段：

- `date`
- `mode`
- `starting_equity_usd`
- `peak_equity_usd`
- `daily_open_count`
- `open_positions`
- `closed_positions`
- `last_scan_time`
- `strategy_stats`

`strategy_stats` 至少保存：

- `traded_volume_est_usd`
- `traded_volume_source`
- `reserve_floor_usd`
- `reserve_buffer_usd`
- `attack_book_usd`
- `reserve_book_usd`
- `last_signal_memory`
- `last_execution_chain`
- `last_status_check_time`

每个 `open_positions[]` 至少记录：

- `chain`
- `lane`
- `token_address`
- `symbol`
- `entry_time`
- `entry_price`
- `entry_usd`
- `quantity`
- `stop_price`
- `highest_price`
- `trail_armed`
- `realize_priority`
- `exit_reason_pending`

## 结构化模式

- `DEFENSE`
  回撤防守，不开新仓
- `RECOVERY`
  只允许极小恢复单，优先修复节奏
- `BASE`
  正常做单，以结构和流动性优先
- `ATTACK`
  只在候选质量和执行条件都足够好时使用

## 双通道框架

### Impulse Lane

目标：捕捉短时间高弹性动量段。

偏好：

- Solana 优先，其次 X Layer
- 中低市值
- 最近 5m 爆量明显
- trigger wallet count 较高
- sold ratio 低
- 更快触发 trailing take profit

特点：

- 持仓更轻
- 止盈更快
- 只保留最强 1-2 笔

### Flow Lane

目标：捕捉更稳的趋势延续和更容易兑现的利润段。

偏好：

- X Layer 优先，其次 Solana
- 流动性更高
- 价格结构更稳
- 更容易低滑点卖出
- 更重视退出可实现性

特点：

- 单笔规模略高于 Impulse Lane
- 更强调“能卖出去”和“落袋为安”

### Whale Relay Trigger

目标：当热点已经形成、但还处于可交易中前段时，允许策略像更真实的交易员一样追进去。

偏好：

- 至少 `3` 个触发钱包，且最好包含 `walletType=3` 的巨鲸信号，或总触发钱包 `>= 4`
- 热点不是纯末端拉升，而是 5m 加速后仍有 15m 延续
- 流动性充足，且最近成交没有明显断层
- top holder、bundle、dev 结构不过分脏

特点：

- 允许追热点，但只追“有确认的热点”
- 更强调“追击位置”，而不是只看“热点名字”
- 默认采用分批试仓，不一次性打满

### Reserve Buffer

目标：给 gas、滑点和后续决策留出空间。

规则：

- 当总权益 `< $20`：不强制设置 reserve，只保留最低 gas
- 当总权益 `>= $20` 且 `< $50`：建议保留 `$5` 机动缓冲
- 当总权益 `>= $50`：建议保留 `15%~20%` 现金或等价缓冲
- `reserve_book_usd` 不用于主动追单
- `attack_book_usd = total_equity - reserve_book_usd`

这不是为了保守，而是为了：

- 防止一笔失误让钱包失去二次决策能力
- 防止滑点、手续费和补仓冲动把优势吃掉

## 自动执行原则

默认允许自动买卖，不需要逐笔等待确认。

但以下情况必须禁止开新仓：

- 当前链不是 `xlayer` / `solana`
- 风险扫描为 `HIGH/CRITICAL`
- 价格冲击 `> 5%`
- 当前 `DEFENSE`
- 已达到最大持仓数
- 已达到每日最大新开仓数
- 钱包扣除 gas 和 reserve 后，攻击仓位不足
- 标的是稳定币 / 主网币 / wrapped 主网币兑换腿

以下情况允许“受控追击”而不是机械跳过：

- 热点仍处于主战链前排
- 至少出现一轮新的巨鲸/聪明钱包共振
- 不是单根极端拉爆，而是回踩后重新放量，或突破后站稳
- 已完成安全补全与 quote 检查

## 硬风控

1. 最大持仓数不超过 `3`
2. 每天最多新开仓 `5` 笔
3. 回撤超过 `15%` 进入 `DEFENSE`
4. `DEFENSE` 下只允许卖出和汇报
5. `RECOVERY` 下默认不新开仓；只有总权益 `>= $20` 时允许 `1` 笔恢复单
6. `HIGH/CRITICAL` 风险代币禁止自动买入
7. 仅做现货，不做杠杆，不做永续
8. 必须保留足够 gas
9. 总权益 `< $3` 时只侦察，不交易
10. 非主战链不允许自动执行

## 执行顺序

每次 heartbeat 唤醒，按这个固定顺序：

1. 刷新钱包状态与余额
2. 更新本地状态文件
3. 先处理已有持仓
4. 再扫描新机会
5. 只从 `Solana / X Layer` 候选里选单
6. 输出固定格式报告

## 第一步：刷新钱包状态

使用：

- `onchainos wallet status`
- `onchainos wallet balance --force`

目的：

- 确认钱包已登录
- 获取总权益和主战链 gas 余额
- 判断当前是否满足最小安全仓位

如果余额接口失败：

- 不自动交易
- 只做只读汇报或已有仓位风险处理

## 第二步：更新模式和账本

计算：

- `daily_pnl_pct = (current_equity - starting_equity_usd) / max(starting_equity_usd, 0.01)`
- `drawdown_pct = (peak_equity_usd - current_equity) / max(peak_equity_usd, 0.01)`

模式判断：

- `DEFENSE`：`drawdown_pct > 15%`
- `RECOVERY`：`daily_pnl_pct <= -3%`
- `ATTACK`：`daily_pnl_pct > 0` 且回撤 `< 5%` 且候选质量强
- 其他：`BASE`

同时维护策略账本：

- `traded_volume_est_usd`
- `attack_book_usd`
- `reserve_book_usd`
- `last_signal_memory`

如果没有精确成交量接口，就用本地已执行成交累计值估算，但必须在输出中标注：

- `traded_volume_source = local_estimate`

## 第三步：信号扫描

### 主扫描

只对主战链扫描：

```bash
onchainos signal list --chain <xlayer|solana> \
  --wallet-type 1,2,3 \
  --min-address-count 2 \
  --max-market-cap-usd 12000000 \
  --min-liquidity-usd 30000 \
  --limit 30
```

### 侦察扫描

可选读取但不执行：

```bash
onchainos signal list --chain <ethereum|bsc> \
  --wallet-type 1,2,3 \
  --min-address-count 2 \
  --max-market-cap-usd 15000000 \
  --min-liquidity-usd 30000 \
  --limit 10
```

侦察链的作用不是交易，而是提供两个新想法：

1. **Narrative Relay**
   如果某个叙事先在 Ethereum / BNB Chain 被连续追逐，而后在 Solana / X Layer 出现同类资产放量，给主战链候选一个“叙事中继加分”

2. **Migration Timing**
   如果侦察链开始衰减，而主战链刚进入放量早期，说明主战链可能正处于更可交易的第二阶段

## 第四步：候选补全

对主战链候选补这些信息：

- `onchainos token price-info`
- `onchainos token advanced-info`
- `onchainos token liquidity`
- `onchainos market kline --bar 5m`
- `onchainos market kline --bar 15m`
- `onchainos market kline --bar 1H`
- `onchainos security token-scan`

## 第五步：筛选规则

### 必选条件

- 链必须是 `solana` 或 `xlayer`
- 合约地址明确
- `riskLevel` 不能是 `HIGH/CRITICAL`
- 非稳定币 / 非主网币 / 非 wrapped 主网币
- `soldRatioPercent <= 35` 视为标准优质入口
- `soldRatioPercent <= 55` 可作为受控追击入口，但必须叠加巨鲸确认、流动性和结构确认
- `vol_mc_ratio >= 1.5` 为标准要求
- `vol_mc_ratio >= 1.1` 只有在流动性更深、巨鲸共振更强时才可放宽
- 最近信号距离现在不超过 `30` 分钟

### 受控追击例外

如果候选满足以下全部条件，则允许略微激进处理，不必死守最保守阈值：

- `triggerWalletCount >= 4`，或出现明确 `walletType=3` 巨鲸追入
- `soldRatioPercent <= 55`
- Solana 流动性建议 `>= $80k`，X Layer 流动性建议 `>= $50k`
- top10 holder 集中度不过分失控，优先 `< 18%`，容忍上限 `< 25%`
- 近 5m 不是单根末端爆冲，而是突破后站稳，或第一次回踩不破关键位
- `security token-scan` 没有高危标签

### Impulse Lane 偏好

- 市值更小
- 最近 5m 爆量更强
- trigger wallet count 更高
- 更适合快速加速
- 更适合“突破后跟随”或“第一次小回踩再追”

### Flow Lane 偏好

- 流动性更深
- 价格结构更稳
- 更容易低滑点卖出
- 更适合做“涨一段就兑现”
- 更适合巨鲸持续吸筹后的延续段

## 第六步：最终打分

至少包含这些维度：

- 信号新鲜度
- trigger wallet count
- sold ratio
- `vol_mc_ratio`
- 5m 爆量强度
- 15m 延续性
- 1H 趋势方向
- holder / dev / bundle 分布
- 风险等级
- 是否更适合 `Impulse Lane`
- 是否更适合 `Flow Lane`

再加两个新过滤器：

1. **Exit Feasibility Score**
   用流动性、最近成交活跃度、top holder 集中度，给“卖出难度”打分。出口差的币，就算涨得快也降权。

2. **Crowding Fade Penalty**
   如果 sold ratio 过高、bundle 占比高、dev rug 历史重、短时价格已经极端扩张，则做拥挤惩罚，防止买在群体出货后段。

再加两个更像真人交易员的判断：

3. **Whale Conviction Score**
   不只看有没有信号，而是看巨鲸/聪明钱包是否连续、是否同向、是否愿意在放量后继续追价。单次信号不够，二次确认更重要。

4. **Chase Timing Score**
   区分“健康追击”和“末端接力”。如果是突破后站稳、缩量回踩不破、再次放量，分数提高；如果是孤立长阳、拉升过陡、成交突然失真，分数降低。

每轮最多只选：

- `1` 个新标的

如果当前已有 2 个高波动仓位，则只允许：

- `0` 或 `1` 个更稳的 Flow Lane 候选

## 第七步：仓位规则

### 总体

- `DEFENSE`: `0`
- `RECOVERY`: `min($5, 8% 权益)`
- `BASE`: `min(attack_book_usd * 0.30, 20% 总权益)`
- `ATTACK`: `min(attack_book_usd * 0.40, 28% 总权益)`

### Lane 修正

- `Impulse Lane`: 更轻，默认乘以 `0.85`
- `Flow Lane`: 更稳，默认乘以 `1.0`
- `Whale Relay Trigger`: 如果是受控追击，首笔先打 `60%` 计划仓位，确认继续走强后再补剩余 `40%`

### Reserve 约束

如果启用了 `Reserve Buffer`：

- 开仓后总权益可用余额不能把 `reserve_book_usd` 明显打穿

## 第八步：执行买入

先：

1. `onchainos swap quote`
2. 再 `onchainos swap execute`

执行原则：

- 优先用稳定币入场
- 不允许把 `SOL/WSOL/OKB/WOKB/稳定币` 本身当作目标交易 thesis
- price impact `> 5%` 禁买
- 热点追击不允许在单根垂直爆拉末端直接一把梭；优先追“突破站稳”或“第一次不破位回踩”
- 如果是 `Whale Relay Trigger`，默认先试仓，再做二段确认加仓

这条不是偏好，而是纪律：

- 真正有 alpha 的地方在“风险资产本身”
- 不能把稳定币和主网币之间的来回兑换当成有效交易

### 加仓规则

只有在以下情况同时满足时，才允许把试仓补成完整仓位：

- 买入后 `5m` 仍站在突破位上方
- 新一轮 signal 里，巨鲸/聪明钱包没有明显撤退
- 价格没有出现放量长上影
- 新 quote 的 price impact 仍在纪律范围内

## 第九步：优先兑现已实现利润

建仓后记录：

- `entry_price`
- `highest_price`
- `trail_armed`
- `lane`
- `realize_priority`

退出条件：

1. 跌破 `stop_price`
2. `trail_armed=true` 且从高点回撤到 `highest_price * 0.90`
3. 5m 动量连续衰减
4. 聪明钱触发密度下降，同时 sold ratio 快速抬升
5. 已获得足够利润并且更适合“先落袋，再换下一笔”
6. 如果是追击单，买入后前 `2` 根 5m K 线无法继续站稳突破区，立即减仓或直接退出
7. 如果巨鲸信号只出现一次、随后没有跟随，优先按“失败追击”处理，不恋战

执行心法：

- 更重视已实现利润，而不是恋战浮盈
- 对小资金来说，连续兑现的小利润往往比死扛一笔大行情更重要

## 第十步：卖出后更新账本

卖出后必须更新：

- `closed_positions`
- `traded_volume_est_usd`
- `last_signal_memory`

如果某一类叙事连续两笔都赚钱：

- 可以提高同叙事候选的 relay 权重

如果某一类叙事连续两笔都失败：

- 下一轮对同叙事默认降权

## 明确不交易的情形

以下情况一律不交易：

- 钱包太小
- 总权益 `< $3`
- 不是 `solana` / `xlayer`
- 候选是稳定币 / 主网币 / wrapped 主网币类对冲腿
- 没有通过流动性和安全门
- 持仓已满
- 当前 `DEFENSE`
- 市场数据不完整

## 配套面板要求

这个 Skill 对应的 dashboard 应优先展示：

1. 当前模式
2. 总权益
3. 当前持仓与剩余仓位
4. reserve 与 attack 可用资金
5. 主战链候选与优先级
6. 每个候选更偏 `Impulse` 还是 `Flow`
7. 最近平仓结果和策略节奏

## 每轮输出格式

每次 heartbeat 或手动运行后，用下面这个结构输出：

- `模式`
- `总权益`
- `今日 PnL`
- `当前持仓`
- `候选标的`
- `本轮动作`
- `原因`
- `下一轮检查时间`

## fallback 规则

### 如果 signal 接口失败

- 不开新仓
- 只处理已有仓位

### 如果 security 接口失败

- `DEFENSE/RECOVERY` 下不新开仓
- `BASE/ATTACK` 下仅允许最强候选继续，并在报告里标记 `security degraded`

### 如果 wallet balance 失败

- 不开新仓
- 只做只读汇报或已有仓位风险处理

### 如果 scout 链数据失败

- 关闭 `Narrative Relay` 加分
- 其余主战链逻辑仍然可运行

## 为什么这个版本更强

1. **只谈策略，不谈口号**
   文档专注于如何找标的、如何下单、如何退出

2. **入口和出口同等重要**
   不是只会追信号，还会评估卖出难度

3. **引入叙事中继**
   不是只看单链热度，而是看叙事是否从侦察链迁移到主战链

4. **把拥挤交易显式降权**
   有助于避开“信号看起来强，但实际已经在末段”的币

5. **依赖链路单一**
   主信息源与交易执行统一走 onchainOS

6. **敢追，但不是盲追**
   对确认过的热点允许受控追击，更像真实交易员，而不是只会错过行情

7. **先试仓，再确认**
   让策略从“机械 yes/no”变成“先小仓验证，再顺势加码”的真实执行风格
