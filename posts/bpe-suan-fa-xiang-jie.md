---
title: 'BPE 算法详解'
date: 2020-02-03 09:19:23
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
### **Byte Pair Encoding**

在 NLP 模型中，输入通常是一个句子，例如 `"I went to New York last week."`，一句话中包含很多单词（token）。传统的做法是将这些单词以空格进行分隔，例如 `['i', 'went', 'to', 'New', 'York', 'last', 'week']`。然而这种做法存在很多问题，例如模型无法通过 `old, older, oldest` 之间的关系学到 `smart, smarter, smartest` 之间的关系。如果我们能使用将一个 token 分成多个 subtokens，上面的问题就能很好的解决。本文将详述目前比较常用的 subtokens 算法 ——BPE（Byte-Pair Encoding）

现在性能比较好一些的 NLP 模型，例如 GPT、BERT、RoBERTa 等，在数据预处理的时候都会有 WordPiece 的过程，其主要的实现方式就是 BPE（Byte-Pair Encoding）。具体来说，例如 `['loved', 'loving', 'loves']` 这三个单词。其实本身的语义都是 "爱" 的意思，但是如果我们以词为单位，那它们就算不一样的词，在英语中不同后缀的词非常的多，就会使得词表变的很大，训练速度变慢，训练的效果也不是太好。BPE 算法通过训练，能够把上面的 3 个单词拆分成 `["lov","ed","ing","es"]` 几部分，这样可以把词的本身的意思和时态分开，有效的减少了词表的数量。算法流程如下：

1. 设定最大 subwords 个数

    V

2. 将所有单词拆分为单个字符，并在最后添加一个停止符 `</w>`，同时标记出该单词出现的次数。例如，`"low"` 这个单词出现了 5 次，那么它将会被处理为 `{'l o w </w>': 5}`
3. 统计每一个连续**字节对**的出现频率，选择最高频者合并成新的 subword
4. 重复第 3 步直到达到第 1 步设定的 subwords 词表大小或下一个最高频的**字节对**出现频率为 1

例如

```
{'l o w </w>': 5, 'l o w e r </w>': 2, 'n e w e s t </w>': 6, 'w i d e s t </w>': 3}

```

出现最频繁的字节对是**`e`**和**`s`**，共出现了 6+3=9 次，因此将它们合并

```
{'l o w </w>': 5, 'l o w e r </w>': 2, 'n e w es t </w>': 6, 'w i d es t </w>': 3}

```

出现最频繁的字节对是**`es`**和**`t`**，共出现了 6+3=9 次，因此将它们合并

```
{'l o w </w>': 5, 'l o w e r </w>': 2, 'n e w est </w>': 6, 'w i d est </w>': 3}

```

出现最频繁的字节对是**`est`**和**`</w>`**，共出现了 6+3=9 次，因此将它们合并

```
{'l o w </w>': 5, 'l o w e r </w>': 2, 'n e w est</w>': 6, 'w i d est</w>': 3}

```

出现最频繁的字节对是**`l`**和**`o`**，共出现了 5+2=7 次，因此将它们合并

```
{'lo w </w>': 5, 'lo w e r </w>': 2, 'n e w est</w>': 6, 'w i d est</w>': 3}

```

出现最频繁的字节对是**`lo`**和**`w`**，共出现了 5+2=7 次，因此将它们合并

```
{'low </w>': 5, 'low e r </w>': 2, 'n e w est</w>': 6, 'w i d est</w>': 3}

```

...... 继续迭代直到达到预设的 subwords 词表大小或下一个最高频的字节对出现频率为 1。这样我们就得到了更加合适的词表，这个词表可能会出现一些不是单词的组合，但是其本身有意义的一种形式

停止符 `</w>` 的意义在于表示 subword 是词后缀。举例来说：`st` 不加 `</w>` 可以出现在词首，如 `st ar`；加了 `</w>` 表明改字词位于词尾，如 `wide st</w>`，二者意义截然不同

### **BPE 实现**

```
import re, collections

defget_vocab(filename):

    vocab = collections.defaultdict(int)

with open(filename, 'r', encoding='utf-8')as fhand:

for linein fhand:

            words = line.strip().split()

for wordin words:

                vocab[' '.join(list(word)) + ' </w>'] += 1

return vocab

defget_stats(vocab):

    pairs = collections.defaultdict(int)

for word, freqin vocab.items():

        symbols = word.split()

for iin range(len(symbols)-1):

            pairs[symbols[i],symbols[i+1]] += freq

return pairs

defmerge_vocab(pair, v_in):

    v_out = {}

    bigram = re.escape(' '.join(pair))

    p = re.compile(r'(?<!\S)' + bigram + r'(?!\S)')

for wordin v_in:

        w_out = p.sub(''.join(pair), word)

        v_out[w_out] = v_in[word]

return v_out

defget_tokens(vocab):

    tokens = collections.defaultdict(int)

for word, freqin vocab.items():

        word_tokens = word.split()

for tokenin word_tokens:

            tokens[token] += freq

return tokens

vocab = {'l o w </w>': 5, 'l o w e r </w>': 2, 'n e w e s t </w>': 6, 'w i d e s t </w>': 3}

# Get free book from Gutenberg

# wget http://www.gutenberg.org/cache/epub/16457/pg16457.txt

# vocab = get_vocab('pg16457.txt')

print('==========')

print('Tokens Before BPE')

tokens = get_tokens(vocab)

print('Tokens: {}'.format(tokens))

print('Number of tokens: {}'.format(len(tokens)))

print('==========')

num_merges = 5

for iin range(num_merges):

    pairs = get_stats(vocab)

ifnot pairs:

break
    best = max(pairs, key=pairs.get)

    vocab = merge_vocab(best, vocab)

    print('Iter: {}'.format(i))

    print('Best pair: {}'.format(best))

    tokens = get_tokens(vocab)

    print('Tokens: {}'.format(tokens))

    print('Number of tokens: {}'.format(len(tokens)))

    print('==========')

```

输出如下

```
==========

Tokens Before BPE

Tokens: defaultdict(<class 'int'>, {'l': 7, 'o': 7, 'w': 16, '</w>': 16, 'e': 17, 'r': 2, 'n': 6, 's': 9, 't': 9, 'i': 3, 'd': 3})

Number of tokens: 11

==========

Iter: 0

Best pair: ('e', 's')

Tokens: defaultdict(<class 'int'>, {'l': 7, 'o': 7, 'w': 16, '</w>': 16, 'e': 8, 'r': 2, 'n': 6, 'es': 9, 't': 9, 'i': 3, 'd': 3})

Number of tokens: 11

==========

Iter: 1

Best pair: ('es', 't')

Tokens: defaultdict(<class 'int'>, {'l': 7, 'o': 7, 'w': 16, '</w>': 16, 'e': 8, 'r': 2, 'n': 6, 'est': 9, 'i': 3, 'd': 3})

Number of tokens: 10

==========

Iter: 2

Best pair: ('est', '</w>')

Tokens: defaultdict(<class 'int'>, {'l': 7, 'o': 7, 'w': 16, '</w>': 7, 'e': 8, 'r': 2, 'n': 6, 'est</w>': 9, 'i': 3, 'd': 3})

Number of tokens: 10

==========

Iter: 3

Best pair: ('l', 'o')

Tokens: defaultdict(<class 'int'>, {'lo': 7, 'w': 16, '</w>': 7, 'e': 8, 'r': 2, 'n': 6, 'est</w>': 9, 'i': 3, 'd': 3})

Number of tokens: 9

==========

Iter: 4

Best pair: ('lo', 'w')

Tokens: defaultdict(<class 'int'>, {'low': 7, '</w>': 7, 'e': 8, 'r': 2, 'n': 6, 'w': 9, 'est</w>': 9, 'i': 3, 'd': 3})

Number of tokens: 9

==========

```

### **编码和解码**

### **编码**

在之前的算法中，我们已经得到了 subword 的词表，对该词表按照字符个数由多到少排序。编码时，对于每个单词，遍历排好序的子词词表寻找是否有 token 是当前单词的子字符串，如果有，则该 token 是表示单词的 tokens 之一

我们从最长的 token 迭代到最短的 token，尝试将每个单词中的子字符串替换为 token。 最终，我们将迭代所有 tokens，并将所有子字符串替换为 tokens。 如果仍然有子字符串没被替换但所有 token 都已迭代完毕，则将剩余的子词替换为特殊 token，如 `<unk>`

例如

```
# 给定单词序列

["the</w>", "highest</w>", "mountain</w>"]

# 排好序的subword表

# 长度 6         5           4        4         4       4          2

["errrr</w>", "tain</w>", "moun", "est</w>", "high", "the</w>", "a</w>"]

# 迭代结果

"the</w>" -> ["the</w>"]

"highest</w>" -> ["high", "est</w>"]

"mountain</w>" -> ["moun", "tain</w>"]

```

### **解码**

将所有的 tokens 拼在一起即可，例如

```
# 编码序列

["the</w>", "high", "est</w>", "moun", "tain</w>"]

# 解码序列

"the</w> highest</w> mountain</w>"

```

### **编码和解码实现**

```
import re, collections

defget_vocab(filename):

    vocab = collections.defaultdict(int)

with open(filename, 'r', encoding='utf-8')as fhand:

for linein fhand:

            words = line.strip().split()

for wordin words:

                vocab[' '.join(list(word)) + ' </w>'] += 1

return vocab

defget_stats(vocab):

    pairs = collections.defaultdict(int)

for word, freqin vocab.items():

        symbols = word.split()

for iin range(len(symbols)-1):

            pairs[symbols[i],symbols[i+1]] += freq

return pairs

defmerge_vocab(pair, v_in):

    v_out = {}

    bigram = re.escape(' '.join(pair))

    p = re.compile(r'(?<!\S)' + bigram + r'(?!\S)')

for wordin v_in:

        w_out = p.sub(''.join(pair), word)

        v_out[w_out] = v_in[word]

return v_out

defget_tokens_from_vocab(vocab):

    tokens_frequencies = collections.defaultdict(int)

    vocab_tokenization = {}

for word, freqin vocab.items():

        word_tokens = word.split()

for tokenin word_tokens:

            tokens_frequencies[token] += freq

        vocab_tokenization[''.join(word_tokens)] = word_tokens

return tokens_frequencies, vocab_tokenization

defmeasure_token_length(token):

if token[-4:] == '</w>':

return len(token[:-4]) + 1

else:

return len(token)

deftokenize_word(string, sorted_tokens, unknown_token='</u>'):

if string == '':

return []

if sorted_tokens == []:

return [unknown_token]

    string_tokens = []

for iin range(len(sorted_tokens)):

        token = sorted_tokens[i]

        token_reg = re.escape(token.replace('.', '[.]'))

        matched_positions = [(m.start(0), m.end(0))for min re.finditer(token_reg, string)]

if len(matched_positions) == 0:

continue
        substring_end_positions = [matched_position[0]for matched_positionin matched_positions]

        substring_start_position = 0

for substring_end_positionin substring_end_positions:

            substring = string[substring_start_position:substring_end_position]

            string_tokens += tokenize_word(string=substring, sorted_tokens=sorted_tokens[i+1:], unknown_token=unknown_token)

            string_tokens += [token]

            substring_start_position = substring_end_position + len(token)

        remaining_substring = string[substring_start_position:]

        string_tokens += tokenize_word(string=remaining_substring, sorted_tokens=sorted_tokens[i+1:], unknown_token=unknown_token)

break
return string_tokens

# vocab = {'l o w </w>': 5, 'l o w e r </w>': 2, 'n e w e s t </w>': 6, 'w i d e s t </w>': 3}

vocab = get_vocab('pg16457.txt')

print('==========')

print('Tokens Before BPE')

tokens_frequencies, vocab_tokenization = get_tokens_from_vocab(vocab)

print('All tokens: {}'.format(tokens_frequencies.keys()))

print('Number of tokens: {}'.format(len(tokens_frequencies.keys())))

print('==========')

num_merges = 10000

for iin range(num_merges):

    pairs = get_stats(vocab)

ifnot pairs:

break
    best = max(pairs, key=pairs.get)

    vocab = merge_vocab(best, vocab)

    print('Iter: {}'.format(i))

    print('Best pair: {}'.format(best))

    tokens_frequencies, vocab_tokenization = get_tokens_from_vocab(vocab)

    print('All tokens: {}'.format(tokens_frequencies.keys()))

    print('Number of tokens: {}'.format(len(tokens_frequencies.keys())))

    print('==========')

# Let's check how tokenization will be for a known word

word_given_known = 'mountains</w>'

word_given_unknown = 'Ilikeeatingapples!</w>'

sorted_tokens_tuple = sorted(tokens_frequencies.items(), key=lambda item: (measure_token_length(item[0]), item[1]), reverse=True)

sorted_tokens = [tokenfor (token, freq)in sorted_tokens_tuple]

print(sorted_tokens)

word_given = word_given_known

print('Tokenizing word: {}...'.format(word_given))

if word_givenin vocab_tokenization:

    print('Tokenization of the known word:')

    print(vocab_tokenization[word_given])

    print('Tokenization treating the known word as unknown:')

    print(tokenize_word(string=word_given, sorted_tokens=sorted_tokens, unknown_token='</u>'))

else:

    print('Tokenizating of the unknown word:')

    print(tokenize_word(string=word_given, sorted_tokens=sorted_tokens, unknown_token='</u>'))

word_given = word_given_unknown

print('Tokenizing word: {}...'.format(word_given))

if word_givenin vocab_tokenization:

    print('Tokenization of the known word:')

    print(vocab_tokenization[word_given])

    print('Tokenization treating the known word as unknown:')

    print(tokenize_word(string=word_given, sorted_tokens=sorted_tokens, unknown_token='</u>'))

else:

    print('Tokenizating of the unknown word:')

    print(tokenize_word(string=word_given, sorted_tokens=sorted_tokens, unknown_token='</u>'))

```

输出如下

```
Tokenizing word: mountains</w>...

Tokenizationof the known word:

['mountains</w>']

Tokenization treating the known wordas unknown:

['mountains</w>']

Tokenizing word: Ilikeeatingapples!</w>...

Tokenizatingof the unknown word:

['I', 'like', 'ea', 'ting', 'app', 'l', 'es!</w>']

```

### **Reference**

- **[3 subword algorithms help to improve your NLP model performance](https://medium.com/@makcedward/how-subword-helps-on-your-nlp-model-83dd1b836f46)**
- **[Tokenizers: How machines read](https://blog.floydhub.com/tokenization-nlp/)**
- **[Overview of tokenization algorithms in NLP](https://towardsdatascience.com/overview-of-nlp-tokenization-algorithms-c41a7d5ec4f9)**
- **[一文读懂 BERT 中的 WordPiece](https://www.cnblogs.com/huangyc/p/10223075.html)**
- **[Byte Pair Encoding](https://leimao.github.io/blog/Byte-Pair-Encoding/)**
- **[深入理解 NLP Subword 算法：BPE、WordPiece、ULM](https://zhuanlan.zhihu.com/p/86965595)**