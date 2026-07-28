# 因子索引库

| 主因子 | 副因子 | 年化 | 收益回撤比 | 概要 | 本地文档 | 原帖 |
|--------|--------|------|-----------|------|---------|------|
| 小市值 | 成交额缩波; 超跌 | 135.34% | 3.43 | 基于总市值×成交额10日STD识别冷门小盘股，叠加回归动量（20日涨跌幅升序）超跌反转，3天轮动 | [查看](../strategy/docs/总市值成交额STD_回归动量选股策略.md) | [原帖](https://bbs.quantclass.cn/thread/87508) |
| 成交额STD+小市值(组合方法论) | 流动性深度; TRIX; 龙头股特征; 净主动买入强度; 成交额稳定性; 穿刺形态 | — | 1.31 | 从200个因子中通过442次回测找最优组合。基础=成交额STD+小市值(Calmar 0.80)，逐步添加至5维度(Calmar 1.31)。发现5因子为甜蜜点、流动性类增量最大、选股数量为隐藏杠杆 | [查看](../strategy/docs/442次多因子组合搜索方法论.md) | [原帖](https://bbs.quantclass.cn/thread/86784) |
| 反转Ret | 小市值; 成交额STD; 极端突破信号改; 倒锤子线信号; 成本分布离散度改; 波动率过滤 | 156.41% | 4.42 | 过拟合示范策略，Ret+小市值+K线形态+多重过滤 | [查看](../strategy/docs/过拟合之基于Ret市值等11因子选股策略.md) | [原帖](https://bbs.quantclass.cn/thread/87621) |
| 反转Ret | 小市值; 成交额STD; 极端突破信号改; 倒锤子线信号; 成本分布离散度改; 上影线/下影线过滤 | 150.88% | 4.21 | 上下影线过滤改版，过拟合示范 | [查看](../strategy/docs/基于小市值上下影线比例等因子选股策略.md) | [原帖](https://bbs.quantclass.cn/thread/87515) |
| 反转Ret | 小市值; 成交额STD; 极端突破信号改; 倒锤子线信号; 成本分布离散度改; 下影线过滤 | 146.22% | 4.0 | 过拟合谱系之源，剔除北交所后仅9% | [查看](../strategy/docs/基于Ret市值等6因子选股策略.md) | [原帖](https://bbs.quantclass.cn/thread/86885) |
| 协整偏离度（超跌） | MACD三线; 成交额稳定性; 量价协同强度; 龙头股特征 | 78.01% | 3.74 | 多维度超跌反弹，排除中小创北 | [查看](../strategy/docs/MCXLL多维度超跌反弹策略.md) | [原帖](https://bbs.quantclass.cn/thread/86906) |
| RankIC代理_20日 | 小市值; 倒锤子线; N字突破上涨; 成交额STD | 90.94% | 1.92 | Allen初版，全升序缝合怪 | [查看](../strategy/docs/缝合怪RankIC代理等5因子策略.md) | [原帖](https://bbs.quantclass.cn/thread/86813) |
| 小市值（26因子缝合） | 量价/动量/K线/横盘等共26因子 | 93.04% | 3.49 | 26因子缝合怪，过拟合警告 | [查看](../strategy/docs/26因子37万净值-过拟合警告.md) | [原帖](https://bbs.quantclass.cn/thread/86883) |
| 小市值 | 成交额缩波; 振幅; Ret; 质量门过滤; 回归择时 | 81.45% | 3.49 | 小市值+质量过滤+择时 | [查看](../strategy/docs/金风重丰-小市值质量过滤择时策略.md) | [原帖](https://bbs.quantclass.cn/thread/87707) |
| 小市值 | Donchian通道位置（择时）; 枢轴点强度; Alpha4/84 | 70.87% | 3.24 | 小市值+Donchian择时 | [查看](../strategy/docs/Seth-市值Donchian枢轴点强度策略.md) | [原帖](https://bbs.quantclass.cn/thread/86842) |
| 小市值 | 成交额STD; 流动性螺旋; 成交量季节性月度 | 49.92% | 3.13 | 4因子全升序低风险 | [查看](../strategy/docs/不甜不放盐-市值成交额STD流动性螺旋策略.md) | [原帖](https://bbs.quantclass.cn/thread/86923) |
| 小市值 | Donchian通道位置（择时）; 枢轴点强度; Alpha4/84 | 71.49% | 3.01 | Seth原版（非预处理）| [查看](../strategy/docs/Seth-非预处理版-市值Donchian枢轴点强度策略.md) | [原帖](https://bbs.quantclass.cn/thread/86765) |
| Bias（自研） | 无 | 71.27% | 2.77 | 自研单因子 | [查看](../strategy/docs/Link-自研单因子Bias策略.md) | [原帖](https://bbs.quantclass.cn/thread/87388) |
| 小市值 | 振幅; RET5; 成交额缩波 | 88.7% | 2.75 | 回调+缩波 | [查看](../strategy/docs/初学-回调缩波策略.md) | [原帖](https://bbs.quantclass.cn/thread/87785) |
| 小市值 | Alpha84 | 79.78% | 2.65 | 小市值+Alpha84 | [查看](../strategy/docs/Hh-alpha84小市值策略.md) | [原帖](https://bbs.quantclass.cn/thread/87088) |
| 小市值 | 20天高点回落 | 59.78% | 2.62 | 高点回落小市值 | [查看](../strategy/docs/John-20天高点回落小市值策略.md) | [原帖](https://bbs.quantclass.cn/thread/86919) |
| Alpha124量价错杀 | 四维确认链 | 99.48% | 2.59 | 量价错杀高收益 | [查看](../strategy/docs/三秒-Alpha124量价错杀四维确认链策略.md) | [原帖](https://bbs.quantclass.cn/thread/87124) |
| 小市值 | 商誉资产占比; 趋势记忆衰减; 换手率 | 66.56% | 2.59 | 基本面+技术面混合 | [查看](../strategy/docs/施塔克-市值商誉趋势记忆换手率策略.md) | [原帖](https://bbs.quantclass.cn/thread/87330) |
| 小市值 | 筛选因子 | 69.2% | 2.58 | 小市值+筛选 | [查看](../strategy/docs/半岛微凉-小市值筛选因子策略.md) | [原帖](https://bbs.quantclass.cn/thread/86823) |
| 小市值 | 云游策略优化 | 91% | 2.57 | 优化云游 | [查看](../strategy/docs/hans郑-优化云游策略.md) | [原帖](https://bbs.quantclass.cn/thread/87487) |
| 国泰君安量价因子池 | 广发LLT择时 | 32.34% | 2.48 | 机构因子+择时低风险 | [查看](../strategy/docs/刘小力-国泰君安量价因子32+4策略.md) | [原帖](https://bbs.quantclass.cn/thread/87867) |
| 小市值（遗传筛选） | 均线支撑; 均线多头排列; 成交额稳定性; CCI; MACD择时 | 45.7% | 2.48 | 遗传算法优化 | [查看](../strategy/docs/游击战策略-遗传算法均线CCI策略.md) | [原帖](https://bbs.quantclass.cn/thread/87018) |
| 小市值 | 放量上涨占比; 资金曲线择时 | 49.41% | 2.46 | 放量上涨+择时 | [查看](../strategy/docs/大苗条-市值放量上涨占比策略.md) | [原帖](https://bbs.quantclass.cn/thread/86825) |
| 小市值 | 涨跌幅; 成交额缩波; ROE; 归母净利润增速; 择时模式切换 | 68% | 2.45 | 基本面+技术面动态切换 | [查看](../strategy/docs/LK-小市值涨跌幅成交额缩波ROE策略.md) | [原帖](https://bbs.quantclass.cn/thread/87241) |
| 小市值 | 量价背离; 布林带择时 | 71.64% | 2.42 | 量价背离+布林带 | [查看](../strategy/docs/Lucian-小市值量价背离布林带择时策略.md) | [原帖](https://bbs.quantclass.cn/thread/86980) |
| 小市值 | 中户散户净买入; MA择时 | 52.26% | 2.39 | 资金流向监控 | [查看](../strategy/docs/K无名-小市值中户散户净买入MA择时策略.md) | [原帖](https://bbs.quantclass.cn/thread/87019) |
