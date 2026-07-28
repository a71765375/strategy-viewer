# 策略详情

**作者**: 三秒

## 策略逻辑

</style><script charset="utf-8" src="/_nuxt/commons/pages/essencethread/_id/pages/essencethread/_id copy/pages/my/notice/pages/thread/_id/pages/~915755c1.f9e73a9.js"></script><script charset="utf-8" src="/_nuxt/commons/ab100162~ff5c4196.c39ca7a.js"></script><script charset="utf-8" src="/_nuxt/commons/33b9d0e8~7274e1de.c3ffb34.js"></script><script charset="utf-8" src="/_nuxt/commons/5a7ee80d~31ecd969.6e449df.js"></script><link rel="preload" as="style" href="/_nuxt/pages/thread/_id~01e7b97c.1a9831

## 因子配置

```python
<br>
('市值', True, None, 0.5),        # 小票方向<br>
('最高价_回撤', True, 120, 1.0),  # 长期超跌确认<br>
('bias', True, 5, 0.5),          # 短期超跌确认<br>
('bias', True, 10, 0.5),         # 中期超跌确认<br>
('Alpha124', True, 20, 0.3),     # 核心：VWAP量价错杀<br>
```

## 策略参数
```python
'hold_period': '3D',                  # 3天一换<br>
'select_num': 2,                     # 2只极端精选<br>
"factor_list": [<br>
('市值', True, None, 0.5),        # 小票方向<br>
('最高价_回撤', True, 120, 1.0),  # 长期超跌确认<br>
('bias', True, 5, 0.5),          # 短期超跌确认<br>
('bias', True, 10, 0.5),         # 中期超跌确认<br>
('A
```
