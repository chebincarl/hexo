---
title: Hash算法和数据库实现
date: 2018-11-25 17:39:24
tags: PHP
---
##  Hash算法与数据库实现
Hash表( HashTable )又称散列表,通过把关键字Key映射到数组中的一个位置来访问记录,以加快查找的速度。这个映射函数称为Hash函数,存放记录的数组称为Hash表。
<!--more-->
### Hash函数
Hash函数的作用是把任意长度的输入,通过Hash算法变换成固定长度的输出,该输出就是Hash值。这种转换是一种压缩映射,也就是Hash值的空间通常远小于输入的空间,不同的输入可能会散列成相同的输出,而不可能从Hash值来唯一地确定输入值。
一个好的Hash函数应该满足以下条件:每个关键字都可以均匀地分布到Hash表任意一个位置,并与其他已被散列到Hash表中的关键字不发生冲突。这是Hash函数最难实现的。

### Hash算法
关键字k可能是整数或者字符串,可以按照关键字的类型设计不同的Hash算法。整数关键字的Hash算法有以下几种。

#### 直接取余法
直接取余法原理比较简单,直接用关键字k除以Hash表的大小m取余数,算法如下:
h (k) =kmodm
例如,如果Hash表的大小为m-12,所给关键字为k=100,则h (k)=4,这种算法只需要一个求余操作,速度比较快。

#### 乘积取整法
乘积取整法首先使用关键字k乘以一个常数A (0<A<1 ),并抽取出kA的小数部分。然后用Hash表大小m乘以这个值,再取整数部分即可。算法如下:
h (k) =floor (m* (KA mod1))
其中, kA mod 1表示kA的小数部分, floor是取整操作。
当关键字是字符串的时候,就不能使用上面的Hash算法。因为字符串是由字符组成,所以可以把字符串所有字符的ASCIl马加起来得到一个整数,然后再按照上面的Hash算法去计算即可。算法如下:
function hash ($key, $m) {
	$strlen = strlen ($key ); 
	$hashval = 0;
	for ($i=0; $i<$strlen; $i++) {
		$hashval += ord ($key {$i});
	);
	return $hashval% $m;
}
虽然这是最简单的Hash算法,而且效果也不好,但可以描述Hash算法的基本原理

#### 经典Hash算法Times33
经过计算机科学家们多年的研究,创造了一些非常有效的Hash算法,比较有名的包括: ELFHash, APHash和Times33等。下面是经典的Times33算法:
unsigned int DJBHash (char"str) {unsigned int hash=5381; while (*str) {hash+= (hash< <5) +(*str++);
।return (hash&Ox7FFFFFFF);।
	Times33算法思路就是不断乘以33,其效率和随机性都非常好,广泛运用于多个开源项目中,如Apache,Perl和PHP等。