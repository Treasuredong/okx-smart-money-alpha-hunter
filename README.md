# okx-smart-money-alpha-hunter
This is a smart money trading skill with onchainOS as its core infrastructure, designed for real-world on-chain execution. Its key features are not aggressiveness, but rather "selectivity in hot topics, discipline in execution, and a balance between profit pursuit and exit quality."

`okx-smart-money-alpha-hunter` 是一个基于 `onchainOS` 的链上现货自动交易 Skill，核心聚焦 `Solana` 和 `X Layer`，通过聪明钱/KOL/巨鲸信号发现高质量机会，并结合流动性、成交结构、持仓分布和风险扫描来决定是否执行交易。它不是简单追热点，而是更强调“能不能赚”和“后面能不能顺利卖掉”。

这个 Skill 的特点主要体现在三点。
第一，结构清晰，包含完整的元数据、触发场景、执行流程、风控规则和输出格式，便于理解和真实运行。

第二，触发和指令设计偏实战，能覆盖“聪明钱策略”“巨鲸信号跟单”“自动扫链上机会”等不同表达，并通过 `Whale Relay Trigger`、受控追击、分批试仓、二段确认加仓等机制，让策略更接近真实交易员的决策方式。

第三，执行上严格以 `onchainOS` 作为主要信息源和交易工具，只在主战链做自动执行，同时保留最大持仓数、日内开仓上限、风险等级过滤、滑点限制和回撤保护，兼顾效率与安全。

这个 Skill 不追求“看到热点就盲冲”，而是强调三件事：
先确认资金流和叙事是否真的形成了可跟随的主升结构
再判断标的是否具备足够好的退出条件，而不是只看买点
最后在严格仓位管理和风控门槛下执行，避免因为小样本情绪或低质量流动性导致失真交易

整体来说，这个 Skill 的优势不只是“发现热点”，而是把信号识别、进场判断、仓位管理和退出质量放进同一个闭环里，形成一个更可执行、也更贴近真实链上交易的自动化策略。
<img width="3002" height="1332" alt="image" src="https://github.com/user-attachments/assets/75ab1039-19ec-4adb-97eb-58cc86c2e207" />
<img width="3022" height="1644" alt="image" src="https://github.com/user-attachments/assets/2b9ec58d-908b-49f7-96f6-1eaa2e969c49" />
<img width="3016" height="1616" alt="image" src="https://github.com/user-attachments/assets/c418b2f6-9ea7-40e5-864a-eb91f74f42d0" />
