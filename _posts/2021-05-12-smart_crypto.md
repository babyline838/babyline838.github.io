---
layout: post
title: WP for smart-cryptooo
---
萌新瑟瑟发抖

Anyway, 鉴于陆续有师傅来问exp，干脆写个wp吧。

https://archive.ooo/c/smart-cryptooo/420/

![image-20210512011142020](/assets/img/smart-crypto.assets/image-20210512011142020.png)

比赛的时候只有这两个文件，当然存档以后为了保证原网页可用性，当时网页的存档也提供了一份在archive里。

加密脚本里训练了一套ANC的模型，然后用Alice对明文做了加密，同时没有给模型的weight，属于灰盒攻击。

Ref: https://mathybit.github.io/adversarial-neural-crypto/

虽然可以自己train一套模型，但weight不一样显然是没法解密密文的，唯密文解密基本上行不通。也不能指望Eve可以解密，因为按照原论文里Eve就是被设计成希望没法唯密文解密的。

(顺便一提这一套需要tf2的环境，tf1训不动)

既然唯密文没办法就只能考虑明文的事情了，同时提供的文件名暗示了明文是当时网站的某个网页。明文加密后的长度和提供的密文差不多，但短一些。相差的部分就是secret了。

___

已知(大部分)明文后，一个自然的想法是用明文-密文对去训练模型，用训练好的模型去解密，这种做法可以看这份wp：

https://github.com/blatchley/defcon-challenge-notes#smart-crypto

当时(发现这题被打了easy的tag以后)在discord里讨的hint是'classical blockcipher tricks and training a model both works'，遂考虑分块加密的问题(当时我魔改了一下anc.py但是它会报segment fault，而且de不出来，作罢)。

队里的dalao提出了关键的猜想：这个模型实际上是线性的(作为老丹师至今无法相信)：

$C=[A~|~B]\left [\begin{matrix}m\\k\end{matrix} \right ]$

当然模型里可能还有奇奇怪怪的bias项：

$C=[A~|~B]\left [\begin{matrix}m\\k\end{matrix} \right ]+bias$

于是只要根据已知的明文和密文去反解$A$和$B$ 就可以了。

---

实现上需要注意的细节：

- 由于奇怪的bias在，尽可能用 $m,c$ 的差值去反解(当时我们没有考虑可能有bias在，所以在模型里考虑进bias应该就不用了)
- 先解$A$，然后用密文里多出来的 ${\rm Enc}(netkey)$ 去反算出$nextkey$
- 有了 $nextkey$ 以后，放进**下一个**bunch里去反解 $B$
- secrect并没有放在明文的最后，而是**插在了当中**，大约103bunch开始，这是第二个难点(所以用所有数据去拟合的话效果反而极差)

具体如何反算，可以调用现成的最小二乘法，我是翻了一遍scipy的文档没找到向量形式的优化包于是用torch搓了一个梯度下降。

![image-20210512015614543](/assets/img/smart-crypto.assets/image-20210512015614543.png)



---

dalao们太强了，而我只是个无情的exp机器人，嘤。