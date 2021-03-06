---
title       : 非参数统计
subtitle    : 单样本检验
author      : yaleidu
job         : 为人民服务
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : prettify  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
github: 
  user: badbye
  repo: Non-parameter.single
---

## 目录:

1.1 广义符号检验

1.2 Wilcoxon符号秩检验]    

1.3 正态记分检验    

1.4 Cox-Stuart趋势检验

1.5 随机游程检验


---
## 1.1 单样本广义符号检验
利用观测值和零假设的中心位置之差的符号来进行检验，忽略差的大小信息．


```r
sign.test = function(x, p, q0) {
    # p:分位点　q0:待检分位数
    n1 = sum(x < q0)
    n2 = sum(x > q0)
    n = n1 + n2
    H1.one = ifelse(as.numeric(quantile(x, p)) > q0, "H1: q > q0", "H1: q < q0")
    p.one = min(pbinom(n1, n, p), 1 - pbinom(n1 - 1, n, p))
    list(H1.OneSide = H1.one, p.OneSide = p.one, H1.TwoSide = "H1: q != q0", 
        p.TwoSide = 2 * p.one)
}
```


--- 
## 1.2 单样本Wilcoxon符号秩检验


在广义符号检验的基础上对数据差异的大小信加以利用息．    
适用范围: **连续对称的总体分布的中位数检验**    


```r
wilcox.single = function(x, m0) {
    m = median(x)
    H1.one = paste(ifelse(m > m0, "H1: M >", "H1: M <"), m0)
    H1.two = paste("H1: M !=", m0)
    d = x - m0
    r = rank(abs(d))
    w.t = sum(r[d > 0])
    w.f = sum(r[d < 0])
    p.one = psignrank(min(w.t, w.f), length(x))
    list(H1.one = H1.one, p.one = p.one, H1.two = H1.two, p.two = 2 * p.one)
}
```


自带函数:**wilcox.test**, 默认进行双边检验       
* exact:精确检验；correct:连续性修正;
* conf.int:是否给出置信区间； conf.level: 置信水平

---

## 1.3 单样本正态记分检验
**较好的大样本性质,善于对正态总体的检验.**

```r
ns.single = function(x, m0) {
    m = median(x)
    n = length(x)
    H1.one = paste(ifelse(m > m0, "H1: M >", "H1: M <"), m0)
    H1.two = paste("H1: M !=", m0)
    d = x - m0
    r = rank(abs(d))
    s = qnorm(0.5 * (1 + r/(1 + n))) * sign(d)
    t = sum(s)/sqrt(sum(s^2))
    p.one = min(pnorm(t), pnorm(t, low = F))
    list(H1.one = H1.one, p.one = p.one, H1.two = H1.two, p.two = 2 * p.one)
}
```

---


## 1.4 Cox-Stuart趋势检验
发展趋势的增减变化检验．


```r
cox.stuart = function(x) {
    n = length(x)
    pairs = floor(n/2)
    D = x[1:pairs] - x[(pairs + 1):n]
    s.t = sum(D > 0)
    s.f = sum(D < 0)
    H1.one = ifelse(s.t > s.f, "H1: Decreasing Trend", "H1: Inceasing Trend")
    H1.two = "H1: Decreasing or Increasing Trend"
    p.one = pbinom(min(s.t, s.f), pairs, 0.5)
    list(H1.one = H1.one, p.one = p.one, H1.two = H1.two, p.two = 2 * p.one)
}
```

---

## 1.5 随机游程检验
检验数据变化是否由随机因素影响．
**默认输出精确检验的双边p值**．

```r
random.run = function(x) {
    if (all(x %in% c(0, 1)) == FALSE) 
        stop("Invalide input")
    if (length(unique(x)) == 1) 
        stop("Maybe something wrong your data?")
    N = length(x)
    m = sum(x == 1)
    n = sum(x == 0)
    k = sum(x[1:(N - 1)] != x[2:N]) + 1
    F = function(r) {
        ifelse(r%%2 == 0, 2 * choose(m - 1, r/2 - 1) * choose(n - 1, r/2 - 1)/choose(N, 
            n), (choose(m - 1, r/2 - 1.5) * choose(n - 1, r/2 - 0.5) + choose(m - 
            1, r/2 - 0.5) * choose(n - 1, r/2 - 1.5))/choose(N, n))
    }
    p = 2 * min(sum(sapply(1:k, F)), sum(sapply(k:N, F)))
    list(H1 = "H1: Data is not random", p = p)
}
```

