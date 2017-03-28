---
title: java里如何正确计算检验和
date: 2016-07-24 00:07:38
tags: Java
categories: 编程语言
---
<Excerpt in index | 首页摘要> 
<!-- more -->
<The rest of contents | 余下全文>

```
byte[] Check_Sum(byte[] Data, int Len) 
{
	byte CheckSum = 0;
	for (int i = 0; i < Len; i++)
	CheckSum += Data[i] & 0xFF;
	return Sum;
}
```

要让java字节参加无符号运算，需要&0xFF，等于不要让最高位变成符号位.
Java没有无符号数据类型。