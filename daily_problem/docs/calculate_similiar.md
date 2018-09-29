产品出了一个奇怪的需求，想通过字符串相似度取匹配城市= =（当然，最后证实通过字符串相似度取判断两个字符串是不是一个城市是不对的！！！）

这里就记录一下我计算字符串(英文字符串)相似度的方法吧～

参考文档：

- [python_levenshtein 的安装和使用](https://www.jianshu.com/p/06370a33e1ee)

### **Levenshtein**

- Levenshtein.hamming(str1, str2)

	计算汉明距离。要求str1和str2必须长度一致。是描述两个等长字串之间对应位置上不同字符的个数。
	
	用法：

		>>> import Levenshtein     
		>>> Levenshtein.hamming('abc', 'cba')
		2
		>>> Levenshtein.hamming('abc', 'def')
		3

- Levenshtein.distance(str1, str2)

	计算编辑距离（也成Levenshtein距离）。是描述由一个字串转化成另一个字串最少的操作次数，在其中的操作包括插入、删除、替换。

	用法：

		>>> Levenshtein.distance('abc', 'ab')
		1
		>>> Levenshtein.distance('cxy', 'ab')
		3

- Levenshtein.ratio(str1, str2)
	
	计算莱文斯坦比。计算公式 r = (sum - ldist) / sum, 其中sum是指str1 和 str2 字串的长度总和，ldist是类编辑距离

	<font color="red">注意：</font>这里的类编辑距离不是`Levenshtein.distance(str1, str2)`所说的编辑距离，`Levenshtein.distance(str1, str2)`中三种操作中每个操作+1，而在此处，删除、插入依然+1，但是替换+2
	这样设计的目的：ratio('a', 'c')，sum=2,按2中计算为（2-1）/2 = 0.5,’a','c'没有重合，显然不合算，但是替换操作+2，就可以解决这个问题。

	用法：

		>>> Levenshtein.ratio('a,cdsf', 'abcd')		      
		0.6

<!-- ### **difflib** -->











































