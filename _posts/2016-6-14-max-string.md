---
layout: post
title: 最长公共子串 
categories: Algorithm
description: 最长公共子串的文本比较算法。
keywords: 算法, Needleman/Wunsch
---

####  本文介绍基于最长公共子串的文本比较算法——Needleman/Wunsch算法。

　　以实例说明：字符串A=kitten，字符串B=sitting

　　那他们的最长公共子串为ittn（注：最长公共子串不需要连续出现，但一定是出现的顺序一致），最长公共子串长度为4。

　　

　　定义：

　　LCS(A,B)表示字符串A和字符串B的最长公共子串的长度。很显然，LSC(A,B)=0表示两个字符串没有公共部分。

　　Rev(A)表示反转字符串A 

　　Len(A)表示字符串A的长度

　　A+B表示连接字符串A和字符串B

 

　　性质：

　　LCS(A,A)=Len(A)

　　LCS(A,"")=0

　　LCS(A,B)=LCS(B,A)

　　0≤LCS(A,B)≤Min(Len(A),Len(B))

　　LCS(A,B)=LCS(Rev(A),Rev(B))

　　LCS(A+C,B+C)=LCS(A,B)+Len(C)

　　LCS(A+B,A+C)=Len(A)+LCS(B,C)

　　LCS(A,B)≥LCS(A,C)+LCS(B,C)

　　LCS(A+C,B)≥LCS(A,B)+LCS(B,C)

 

　　为了讲解计算LCS(A,B)，特给予以下几个定义 

　　A=a1a2……aN，表示A是由a1a2……aN这N个字符组成，Len(A)=N

　　B=b1b2……bM，表示B是由b1b2……bM这M个字符组成，Len(B)=M

　　定义LCS(i,j)=LCS(a1a2……ai,b1b2……bj)，其中0≤i≤N，0≤j≤M

　　故：　　LCS(N,M)=LCS(A,B)

　　　　　　LCS(0,0)=0

　　　　　　LCS(0,j)=0

　　　　　　LCS(i,0)=0

 

　　对于1≤i≤N，1≤j≤M，有公式一

　　若ai=bj，则LCS(i,j)=LCS(i-1,j-1)+1

　　若ai≠bj，则LCS(i,j)=Max(LCS(i-1,j-1),LCS(i-1,j),LCS(i,j-1))

 

　　计算LCS(A,B)的算法有很多，下面介绍的Needleman/Wunsch算法是其中的一种。和LD算法类似，Needleman/Wunsch算法用的都是动态规划的思想。在Needleman/Wunsch算法中还设定了一个权值，用以区分三种操作（插入、删除、更改）的优先级。在下面的算法中，认为三种操作的优先级都一样。故权值默认为1。

　　

　　举例说明：A=GGATCGA，B=GAATTCAGTTA，计算LCS(A,B)
![这里写图片描述](http://img.blog.csdn.net/20160614093752767)
![这里写图片描述](http://img.blog.csdn.net/20160614093629906)
![这里写图片描述](http://img.blog.csdn.net/20160614093703860)

　则，LCS(A,B)=LCS(7,11)=6

　　可以看出，Needleman/Wunsch算法实际上和LD算法是非常接近的。故他们的时间复杂度和空间复杂度也一样。时间复杂度为O(MN)，空间复杂度为O(MN)。空间复杂度经过优化，可以优化到O(M)，但是一旦优化就丧失了计算匹配字串的机会了。由于代码和LD算法几乎一样。这里就不再贴代码了。

　　

　　还是以上面为例A=GGATCGA，B=GAATTCAGTTA，LCS(A,B)=6

　　他们的匹配为：

![这里写图片描述](http://img.blog.csdn.net/20160614094353097)

　　如上面所示，蓝色表示完全匹配，黑色表示编辑操作，_表示插入字符或者是删除字符操作。如上面所示，蓝色字符有6个，表示最长公共子串长度为6。

　　利用上面的Needleman/Wunsch算法矩阵，通过回溯，能找到匹配字串

　　第一步：定位在矩阵的右下角
![这里写图片描述](http://img.blog.csdn.net/20160614094556207)

若ai≠bj，回溯到左上角、上边、左边中值最大的单元格，若有相同最大值的单元格，优先级按照左上角、上边、左边的顺序

![这里写图片描述](http://img.blog.csdn.net/20160614094711043)

若当前单元格是在矩阵的第一行，则回溯至左边的单元格

若当前单元格是在矩阵的第一列，则回溯至上边的单元格

![这里写图片描述](http://img.blog.csdn.net/20160614094758114)

依照上面的回溯法则，回溯到矩阵的左上角

　　第三步：根据回溯路径，写出匹配字串

　　　　若回溯到左上角单元格，将ai添加到匹配字串A，将bj添加到匹配字串B

　　　　若回溯到上边单元格，将ai添加到匹配字串A，将_添加到匹配字串B

　　　　若回溯到左边单元格，将_添加到匹配字串A，将bj添加到匹配字串B

　　　　搜索晚整个匹配路径，匹配字串也就完成了

 

　　LD算法和Needleman/Wunsch算法的回溯路径是一样的。这样找到的匹配字串也是一样的。

