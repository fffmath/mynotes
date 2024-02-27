---
title: Public key tools
tags:
  - crypto
hide:
comments: false  
---

首先考虑一个简单的问题：两方想要传送一个共享的秘密值，使得passive eavesdropping adversary获取不到对应的值。这里的passive eavesdropping adversary指的是能够抓取信息但是不能注入/修改该消息。然而在现实生活中需要对后者做出考虑，也就是说需要能够抵抗对消息做手脚的敌手

## Anonymous key exchange
本书是现代密码学的教材，因此采用基于game的语言去描述/定义安全性。
**Attack Game 10.1** (Anonymous key exchange)

对于一个密钥交换协议 $P = (A, B)$，以及给定的对手 $A$，攻击游戏的运行如下：

- 挑战者在 $A$ 和 $B$ 之间运行协议以生成共享密钥 k 和转录 $T_P$。它将 $T_P$ 提供给 $A$。
- $A$ 输出对 $k$ 的猜测 $\hat{k}$。

我们定义 $A$ 的优势，表示为 $AnonKEadv[A, P]$，为 $\hat{k} = k$ 的概率。

**Definition 10.1** 我们说一个匿名密钥交换协议 (Anonymous key exchange) $P$ 对于窃听者是安全的，如果对于所有高效的对手 $A$， $AnonKEadv[A, P]$ 是可以忽略不计的。

这个安全性定义非常弱，有三个原因。首先，我们假设对手无法篡改消息。其次，我们只保证对手无法完全猜出 $k$。这并不排除对手能够猜出 $k$ 的一半比特的可能性。如果我们要将 $k$ 用作秘密会话密钥，我们真正希望的性质是 $k$ 与真正的随机密钥不可区分。第三，该协议未提供参与者身份的保证。我们将在第 21 章加强定义 10.1，以满足这些更强的要求。

考虑到我们在第一部分开发的所有工具，自然而然地会问是否可以使用任意安全的对称密码进行匿名密钥交换。答案是肯定的，正如我们在第 10.8 节中展示的那样，但是生成的协议非常低效。要开发高效的协议，我们首先必须引入一些新的工具。

## One-way trapdoor functions
**Definition 10.2**
- x和y所在的集合不一定一样，当x和y所在的集合一样的时候对应的One-way function实际上是permutation (置换)。
## A trapdoor permutation scheme baesd on RSA
目前RSA基本上是唯一已知的合理的候选trapdoor permutation方案, 还有一些其他方案，但它们可以看作是RSA的变体。

对于每个固定的 $pk = (n, e)$，函数 $F(pk, \cdot)$ 将 $\mathbb{Z}_n$ 映射到 $\mathbb{Z}_n$；因此，这个函数的定义域和值域实际上随 $pk$ 变化。然而，在我们对trapdoor permutation scheme的定义中，不允许函数的定义域和值域随公钥变化而变化。因此，事实上，这个方案并不完全满足trapdoor permutation scheme的形式语法要求。但是可以很容易地推广陷阱置换方案的定义，以允许这种情况发生。在这里我们不打算这样做而是通过下面的习题探讨了一种基于RSA构建适当的trapdoor permutation scheme的想法。

**Exercise (A proper trapdoor permutation scheme based on RSA)**
如上述所讨论的，我们基于RSA的陷阱置换方案并不完全满足我们的定义，简单地因为它作用的域随公钥而变化。现在设 $l$ 和 $e$ 是用于RSA密钥生成的参数，设 $G$ 是密钥生成算法，输出一对 $(pk, sk)$。回忆一下，$pk = (n, e)$，其中 $n$ 是RSA模数，是两个 $l$-bit 素数的乘积，$e$ 是加密指数。密钥是 $sk = (n, d)$，其中 $d$ 是对应于加密指数 $e$ 的解密指数。选择一个参数 $L$，它远远大于 $2l$，使得 $n/2^L$ 是可以忽略不计的。令 $X$ 是范围为 $[0, 2^L)$ 内的整数集合。我们将呈现一个陷阱置换方案 $(G, F^{*}, I^{*})$，定义在 $X$ 上。函数 $F^{*}$ 接受两个输入：如上所述的公钥 $pk$ 和一个整数 $x \in X$，并输出一个整数 $y \in X$，计算如下。将 $x$ 除以 $n$，得到整数商 $Q$ 和余数 $R$，使得 $x = nQ + R$，且 $0 \leq R < n$。如果 $Q > 2^L/n - 1$，则设置 $S := R$；否则，设置 $S := R^e \mod n$。最后，设置 $y := nQ + S$。

- (a) 证明 $F^*(pk, \cdot)$ 是 $X$ 上的一个置换，并给出一个满足对所有 $x \in X$ 都有 $I^*(sk, F^*(pk, x)) = x$ 的高效的求逆的函数 $I^{*}$。

实际上该方案把之前的$\mathbb{Z}_n$ 映射到 $\mathbb{Z}_n$变成了$[0, 2^L)$映射到了$[0, 2^L)$。它是以$n$长度划分区间段来做到这件事情，因此我们的$I^{*}$就可以用类似的想法求出来。将 $y$ 除以 $n$，得到整数商 $Q$ 和余数 $R$，使得 $y := nQ + S$, 如果 $Q > 2^L/n - 1$，则设置 $R := S$；否则，设置 $R := S^d \mod n$。最后，设置 $x := nQ + R$。

- (b) 证明在RSA假设下，$(G, F^{*}, I^{*})$ 是单向的。

反证法，如果不是One-way的话，那么存在一个敌手$\mathcal{A}$,使得$Pr[\mathcal{A}(y)=x]\geq \epsilon$。我们接下来用$\mathcal{A}$来构造可以攻击RSA的算法$\mathcal{A^{\prime}}$：

对于 $y \in \mathbb{Z}_n$, 将y提升到$\mathbb{Z}$上，接下来$\mathcal{A^{\prime}}$调用$\mathcal{A}$，可以得到$Pr[\mathcal{A}(y)=x]\geq \epsilon$，同时$x\in [0,n)$，可以看作为$\mathbb{Z}_n$的元素。



