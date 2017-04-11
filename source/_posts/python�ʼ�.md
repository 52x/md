title: python 学习笔记
date: 2016/07/10
tags: [python, 笔记]

---

在集合中没有重复元素的情况下，考虑使用 set 集合，而非 list，set 的一些常见运算速度比 list 快很多。

对于 for 循环中的迭代，建议使用如下的方式
 
	for i in xrange(1000): pass

**而不是**，如下这种方式：
	
	for i in range(1000): pass

因为第二种方式会生成一个 1000 个元素的 list， 而第一种则不会生成 1000 个元素的 list，而是在每次迭代中返回下一个数值，内存空间占用很小。因为 xrange 不会返回 list，而是返回一个 iterator 对象。

通过 yield 可以生成 iterator 对象，降低内存使用。

例如，我们要对一个文件进行读取，如果直接使用 read() 方法，会导致不可预测的内存占用。好的方法是利用固定长度的缓冲区来不断读取文件内容。通过 yield， 我们不需要编写读取文件的迭代类，就可以轻松实现文件读取：

```
def read_file(path):
	BLOCK_SIZE = 1024
	with open(path, 'rb') as f:
		while True:
			block = f.read(BLOCK_SIZE)
			if blokc:
				yield block
			else:
				return
``` 