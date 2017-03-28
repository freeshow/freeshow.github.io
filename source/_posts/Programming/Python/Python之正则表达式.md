---
title: Python之正则表达式
toc: false
date: 2016-07-24 20:49:06
tags: 
- Python
categories:
- 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

## Python里的正则表达式
- pattern：匹配模式，遵循正则表达式语法
- method：匹配方法，search/match/findall/finditer/sub/subn


## 一个例子，提取价格

```
# -*- coding: utf-8 -*-
import re

def re_demo():
    str = 'If you purchase more than 100 sets,the price of product A is $9.90.'
    #解析数量和价格
    m = re.search(r'(\d+).*\$(\d+\.?\d*)',str)
    print m.groups() #('100', '9.90')
    print m.group(0) #100 sets,the price of product A is $9.90
    print m.group(1) #100
    print m.group(2) #9.90

if __name__ == '__main__':
    re_demo()
```

## re模块介绍

- re.search：搜索字符串，找到匹配的第一个字符串
- re.match：从字符串开始，开始匹配
- search vs match
    - search: 搜索字符串任意位置的匹配
    - 只从字符串的起始位置开始匹配
- split：使用正则表达式来分割字符串
- findall：根据正则表达式从左到右搜索匹配项，返回匹配的字符串列表。
- finditer: 根据正则表达式从左到右搜索匹配项，返回一个迭代器，迭代返回MathObject。
- sub 字符串替换
    - pattern: 正则表达式
    - repl: 替换项，字符串或函数
    - string: 待处理的字符串
- subn与sub一样，返回值多了替换的字符串个数。


#### 实例


```
# -*- coding: utf-8 -*-
import re

def re_demo():

    #search vs match
    s = 'abcd'
    print re.search(r'c',s) #<_sre.SRE_Match object at 0x000000000283A510>，匹配成功
    print re.match(r'c',s) #None，匹配不成功，以为match是从字符串的第一字符开始匹配
    print re.match(r'.*c',s) ##<_sre.SRE_Match object at 0x000000000283A510>，匹配成功
    
     #split
    s = 'There is an apple'
    print re.split(r'\W',s) #['There', 'is', 'an', 'apple']

    #findall
    s = 'There is an apple'
    print re.findall(r'\w+',s) #['There', 'is', 'an', 'apple']
    s = 'The first price is $9.90 and the second price is $100'
    print re.findall(r'\d+.?\d*',s) #['9.90', '100']

    #finditer
    s = 'The first price is $9.90 and the second price is $100'
    ms = re.finditer(r'\d+.?\d*', s)
    for m in ms:
        print m.group()
    #9.90
    #100

    #sub vs subn
    s = 'The first price is $9.90 and the second price is $100'
    print re.sub(r'\d+.?\d*','<number>',s)
    #The first price is $<number> and the second price is $<number>
    print re.subn(r'\d+.?\d*', '<number>', s)
    #('The first price is $<number> and the second price is $<number>', 2)

if __name__ == '__main__':
    re_demo()
```

## MatchObject
能匹配的正则表达式时，返回re.MatchObject

- group():返回匹配的组
    - 索引0表示全部匹配的字符串
    - 索引1开始表示匹配的子组
    - 参数可以一个也可以多个
    - 命名组
    
- groupdict():返回配陪的子组
- groups()：返回配陪的子组，索引从1开始的左右子组
- start/end/span: 返回配陪的位置



```
# -*- coding: utf-8 -*-
import re
def re_demo():
    #group: 正则表达式匹配的结果默认增加一个group，即匹配的整个结果
    s = 'group1 group2'
    m = re.match(r'(\w+) (\w+)+',s)
    print m.group() #group1 group2 ，整个匹配结果为默认的group,也即group(0)
    print m.group(0) #group1 group2
    print m.group(1) #group1
    print m.group(2) #group2
    print m.group(0,1,2) #('group1 group2', 'group1', 'group2')

    #groups()：返回所有的子匹配项
    print  m.groups() #('group1', 'group2')


if __name__ == '__main__':
    re_demo()
```

## 介绍几个特殊的


```
# -*- coding: utf-8 -*-
import re
def re_demo():
    #dot(.):匹配任意字符
    print re.match(r'.*','abc\ndef').group() #abc 匹配1行
    print re.match(r'.*','abc\ndef',re.DOTALL).group()  #re.DOTALL表示dot可以匹配 \n
    # abc
    # def

    #caret(^)
    print re.findall(r'^abc','abc\nabc') #['abc']
    print re.findall(r'^abc','abc\nabc',re.MULTILINE) #['abc','abc']

    #$
    print re.findall(r'abc\d$', 'abc1\nabc2')  # ['abc2']
    print re.findall(r'abc\d$', 'abc1\nabc2',re.MULTILINE)  # ['abc1','abc2']

    #greedy/non-greedy
    s = '<H1>title</H1>'
    print re.match(r'<.*>',s).group() #<H1>title</H1>,贪婪模式匹配(默认)
    print re.match(r'<.*?>',s).group() #<H1>,加?,非贪婪模式匹配
    print re.match(r'ab{2,4}','abbbbb').group() #abbbb
    print re.match(r'ab{2,4}?','abbbbb').group() #abb

    #[]:集合,匹配集合中的一个字符
    print re.search(r'0[xX]([0-9A-Fa-f]{6})','The hex value is 0xFF03D6').group(1) #FF03D6


if __name__ == '__main__':
    re_demo()
```

## re.compile()

- re.compile()的返回类型为RegexObject.
- RegexObject的方法：
    - search
    - match
    - findall
    - split
    - finditer
    - sbu
- 可大大提高效率


### 实例
```
# -*- coding: utf-8 -*-
import re
def re_demo():
    #\number:前一个匹配组
    print re.search(r'(\d)(\d)(\d)\1\2\3','135135').groups() #('1', '3', '5')
    print re.search(r'(\d)(\d)(\d)\1\2\3','135136') #None,不匹配
    print re.search(r'(\d{3})\1','135135135').groups() #('135',)
    print re.search(r'(\d{3})\1','135136') #None
    print re.search(r'(\d{3}) \1','135 135').groups() #('135',)
    print re.search(r'(\d{3}) \1','135 136') #None，不匹配

    #re.VERROSE/re.compile():正则表达式中可添加注释
    s = 'the number is 20.5'
    r = re.compile(r'''
                    \d+ #整数部分
                    \.? #小数点，可能包含也可能不包含
                    \d* #小数部分，可选
                    ''',re.VERBOSE)
    print re.search(r,s).group() #20.5
    #re.compile()
    print r.search(s).group() #20.5

if __name__ == '__main__':
    re_demo()
```










