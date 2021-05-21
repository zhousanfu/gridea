---
title: 'DFA 算法实现敏感词过滤'
date: 2021-05-07 18:04:24
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
**一、DFA 算法简介**

在实现文字过滤的算法中，DFA是唯一比较好的实现算法。

DFA 全称为：Deterministic Finite Automaton，即确定有穷自动机。其特征为：有一个有限状态集合和一些从一个状态通向另一个状态的边，每条边上标记有一个符号，其中一个状态是初态，某些状态是终态。但不同于不确定的有限自动机，DFA 中不会有从同一状态出发的两条边标志有相同的符号。

![https://nos.netease.com/yidun/5c517db5-d268-455e-a98f-338b93d5ed1e.png](https://nos.netease.com/yidun/5c517db5-d268-455e-a98f-338b93d5ed1e.png)

简单点说就是，它是是通过 event 和当前的 state 得到下一个 state，即 event + state= nextstate。理解为系统中有多个节点，通过传递进入的 event，来确定走哪个路由至另一个节点，而节点是有限的。

**二、DFA 算法实践敏感词过滤**

**1. 敏感词库构造**

以王八蛋和王八羔子两个敏感词来进行描述，首先构建敏感词库，该词库名称为SensitiveMap，这两个词的二叉树构造为：

![https://nos.netease.com/yidun/c30653b6-8728-432e-aeae-e95ed54ed8d6.png](https://nos.netease.com/yidun/c30653b6-8728-432e-aeae-e95ed54ed8d6.png)

用 hash 表构造为：

![https://nos.netease.com/yidun/4fcfd09b-7457-4eca-b4f6-bc1ea509065f.png](https://nos.netease.com/yidun/4fcfd09b-7457-4eca-b4f6-bc1ea509065f.png)

怎么用代码实现这种数据结构呢？

![https://nos.netease.com/yidun/33980271-6c5f-4cfd-ab85-5f150555ecf9.png](https://nos.netease.com/yidun/33980271-6c5f-4cfd-ab85-5f150555ecf9.png)

**2. 敏感词过滤**

以上面例子构造出来的 SensitiveMap 为敏感词库进行示意，假设这里输入的关键字为：王八不好，流程图如下：

![https://nos.netease.com/yidun/8d85693e-e6c5-43ca-91ba-bc9e34e6577a.jpg](https://nos.netease.com/yidun/8d85693e-e6c5-43ca-91ba-bc9e34e6577a.jpg)

怎么用代码实现这个流程图逻辑呢？

![https://nos.netease.com/yidun/cec0241a-8fec-46b1-aca5-a616b2a98cd5.png](https://nos.netease.com/yidun/cec0241a-8fec-46b1-aca5-a616b2a98cd5.png)

**三、优化思路**

对于“王*八&&蛋”这样的词，中间填充了无意义的字符来混淆，在我们做敏感词搜索时，同样应该做一个无意义词的过滤，当循环到这类无意义的字符时进行跳过，避免干扰。

来源：博客园   作者：JMCui

原文链接：https://www.cnblogs.com/jmcui/p/11925777.html

【声明】文章来源于网上采集整理，版权归原作者所有，如有侵权，请邮件反馈yidunmarket@126.com，我们将尽快核实修改。

```python
# -*- coding:utf-8 -*-

import time
time1=time.time()

# DFA算法
class DFAFilter():
    def __init__(self):
        self.keyword_chains = {}
        self.delimit = '\x00'

    def add(self, keyword):
        keyword = keyword.lower()
        chars = keyword.strip()
        if not chars:
            return
        level = self.keyword_chains
        for i in range(len(chars)):
            if chars[i] in level:
                level = level[chars[i]]
            else:
                if not isinstance(level, dict):
                    break
                for j in range(i, len(chars)):
                    level[chars[j]] = {}
                    last_level, last_char = level, chars[j]
                    level = level[chars[j]]
                last_level[last_char] = {self.delimit: 0}
                break
        if i == len(chars) - 1:
            level[self.delimit] = 0

    def parse(self, path):
        with open(path,encoding='utf-8') as f:
            for keyword in f:
                self.add(str(keyword).strip())

    def filter(self, message, repl="*"):
        message = message.lower()
        ret = []
        start = 0
        while start < len(message):
            level = self.keyword_chains
            step_ins = 0
            for char in message[start:]:
                if char in level:
                    step_ins += 1
                    if self.delimit not in level[char]:
                        level = level[char]
                    else:
                        ret.append(repl * step_ins)
                        start += step_ins - 1
                        break
                else:
                    ret.append(message[start])
                    break
            else:
                ret.append(message[start])
            start += 1

        return ''.join(ret)

if __name__ == "__main__":
    gfw = DFAFilter()
    path="F:/文本反垃圾算法/sensitive_words.txt"
    gfw.parse(path)
    text="新疆骚乱苹果新品发布会雞八"
    result = gfw.filter(text)

    print(text)
    print(result)
    time2 = time.time()
    print('总共耗时：' + str(time2 - time1) + 's')
```

测试结果:

1) 敏感词 100个

- ---------------dfa-----------**message*** 2240.325479984283-----------normal--------------**message*** 224The count of word: 1000.107350111008

2) 敏感词 1000 个

- ---------------dfa-----------**message*** 2240.324251890182-----------normal--------------**message*** 224The count of word: 10001.05939006805

从上面的实验我们可以看出，在DFA 算法只有在敏感词较多的情况下，才有意义。在百来个敏感词的情况下，甚至不如普通算法

下面从理论上推导时间复杂度，为了方便分析，首先假定消息文本是等长的,长度为lenA；每个敏感词的长度相同，长度为lenB，敏感词的个数是m。

1) DFA算法的核心是构建一棵多叉树，由于我们已经假设，敏感词的长度相同，所以树的最大深度为lenB，那么我们可以说从消息文本的某个位置（字节)开始的某个子串是否在敏感词树中，最多只用经过lenB次匹配.也就是说判断一个消息文本中是否有敏感词的时间复杂度是lenA * lenB

2) 再来看看普通做法，是使用for循环，对每一个敏感词，依次在消息文本中进行查找，假定字符串是使用KMP算法,KMP算法的时间复杂度是O(lenA + lenB)

那么对m个敏感词查找的时间复杂度是 (lenA + lenB ) * m

综上所述，DFA 算法的时间复杂度基本上是与敏感词的个数无关的。