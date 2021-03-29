---
title: '数据清洗'
date: 2021-01-03 12:17:54
tags: [数据挖掘]
published: true
hideInList: false
feature: 
isTop: false
---
- **文本过滤**

    ```python
    def review_to_words(data):
        
        # 正则去除表情
        emoji_pattern = re.compile(u'[\U00010000-\U0010ffff]')
        data = emoji_pattern.sub(u'', data)
        
        # 正则去除标点
        fuhao_pattern = re.compile(u'\.*')
        data = fuhao_pattern.sub(u'', data)
        
        # 正则去除数字
        digit_pattern = re.compile(u'\d+')
        data = digit_pattern.sub(u'', data)
        
        # 空格拆分词语
        words = data.lower().split()
        
        # 去掉rmSignal
        meaningful_words = [w for w in words if not w in rmSignal]
        
        # 将筛分好的词合成一个字符串，并用空格分开
        words = " ".join(meaningful_words)
        return words
    ```