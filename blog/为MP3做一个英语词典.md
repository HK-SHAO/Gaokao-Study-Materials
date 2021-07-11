---
title: 为MP3做一个英语词典
tags:
  - Python
  - 英语
  - mp3
  - 高中
  - 高考
abbrlink: 5d2efed2
date: 2019-09-12 19:29:26
---

# 前言

在我们学校，几乎所有同学都有MP3用来听歌和看小说，但市面上所流行的大多数MP3都没有英语词典功能。同学们往往选择英语词典进行查单词，最流行的牛津高阶词典词量多，但太厚太笨重，查词并不方便。我发现大多数时候，我们查词仅仅是看中文释义、变形或者音标，于是我萌生了个念头，能不能在MP3上做一个查单词的程序？

# 想法

比想象中的要困难。虽然我会一点C语言，也了解一点硬件开发，但基于MP3的芯片（炬力）开发软件，真的太过麻烦。

有一天晚上，在学校宿舍，我突然想到了一个简单的解决办法：将所有单词的字母顺序创建为文件夹，在相应的的文件夹里放一个解释这个单词的txt文件，不就可以实现这个功能了吗？

![37](/images/blog/37.PNG)

![38](/images/blog/38.gif)

我只需要在MP3文件夹视图中，一层层打开相应的文件夹来定位这个单词，得到单词释义。

程序很好写，数据也很好找，放假回家马上就实现了。总共制作了近四万个单词（包括文件夹有十几万），效果如下：

![39](/images/blog/39.gif)

# 成果

把这些文件夹放到一张SD卡里，插到MP3上。打开，效果跟预想中的一样。我在学校试用了2个星期，感觉查单词挺方便的。后来又回家做了个40万单词的版本，在MP3上的最终效果如下图：

![40](/images/blog/40.gif)

# 代码

```python
import csv
import os
import re

def fi(l):
    return re.search(r'\d|\W', l[0]) is None

dic = []
words = []
with open('ecdict.csv','r', encoding='utf-8') as csvfile:
    reader = csv.reader(csvfile)
    for w in reader:
        if fi(w):
            words.append(w[0])
            dic.append(w)

def fo(s):
    n = 0
    for w in words:
        if s == w[:len(s)]:
            n += 1
    if n < 13:
        for w in words:
            if s == w[:len(s)]:
                del w
        return True
    return False

for line in dic:
    w = line[0]
    l3r = ''
    if line[3] is not '':
        l3r = '\n' + line[3].replace('\\n', '；')
    l2r = ''
    if line[2] is not '':
        l2r = '\n英文释义：{}'.format(line[2].replace('\\n', '；').replace('：n ', '：n. ').replace('；n ', '；n. '))
    l10r = ''
    if line[10] is not '':
        l10r = '\n' + line[10].replace('p:', '过去式：').replace('d:', '过去分词：').replace('i:', '现在分词：').replace('r:', '比较级：').replace('t:', '最高级：').replace('s:', '复数：').replace('3:', '三单：').replace('0:', '原型：').replace('1:', '形式：').replace('/', '；')
    l7r = ''
    if line[7] is not '':
        l7r = '\n考试要求：' + line[7].replace('zk', '中考').replace('gk', '高考').replace('cet4', '四级').replace('cet6', '六级').replace('ky', '考研').replace('ielts', '雅思').replace('toefl', '托福').replace('gre', 'GRE').replace(' ', '；')
    l1r = ''
    if line[1] is not '':
        l1r = '\n/{}/'.format(line[1])
    string = '【{}】{}{}{}{}{}'.format(w, l1r, l3r, l10r, l2r, l7r)

    path = ''
    b = ''
    for l in w:
        b += l
        path += '{}-/'.format(b)
        if fo(b):
            break
    path = 'DIC/' + path.lower()
    if not os.path.exists(path):
        os.makedirs(path)
        print(path)
    with open('{}-{}.txt'.format(path, w),'w', encoding='utf-8') as f:
        f.write(string)
    # print(string)
```

### 源代码&使用方法

开源地址：[https://github.com/HK-SHAO/MP3DIC](https://link.zhihu.com/?target=https%3A//github.com/HK-SHAO/MP3DIC)

使用方法很简单，你可以直接在上面的网页内下载 dic.zip 这个文件，解压缩到mp3内即可（文件目录很多，可能要解压缩很久）。