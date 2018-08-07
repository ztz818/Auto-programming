# Auto-programming
结合深度学习技术和枚举搜索自动生成解决竞赛风格的可执行代码<br>
项目用到的程序语言示例：<br>

 ![image](https://github.com/ztz818/Auto-programming/blob/master/2.jpg)


项目用到的程序指令集：

HEAD :: [int] -> int<br>
Given an array, returns its first element (or NULL if the array is empty).<br>
LAST :: [int] -> int<br>
Given an array, returns its last element (or NULL if the array is empty).<br>
TAKE :: int -> [int] -> int<br>
Given an integer n and array xs, returns the array truncated after the n-th element.<br>
DROP :: int -> [int] -> int<br>
Given an integer n and array xs, returns the array with the first n elements dropped <br>
ACCESS :: int -> [int] -> int<br>
Given an integer n and array xs, returns the (n+1)-st element of xs.<br>
MINIMUM :: [int] -> int<br>
Given an array, returns its minimum (or NULL if the array is empty).<br>
MAXIMUM :: [int] -> int<br>
Given an array, returns its maximum (or NULL if the array is empty).<br>
REVERSE :: [int] -> [int]<br>
Given an array, returns its elements in reversed order.<br>
SORT :: [int] -> [int]<br>
Given an array, return its elements in non-decreasing order.<br>
SUM :: [int] -> int<br>
Given an array, returns the sum of its elements. (The sum of an empty array is 0.)<br>

Higher-order functions:<br>

MAP :: (int -> int) -> [int] -> [int]<br>
Given a lambda function f mapping from integers to integers, and an array xs, returns the array resulting from applying f to each element of xs.<br>
FILTER :: (int -> bool) -> [int] -> [int]<br>
Given a predicate f mapping from integers to truth values, and an array xs, returns the elements of xs satisfying the predicate in their original order.<br>
COUNT :: (int -> bool) -> [int] -> int<br>
Given a predicate f mapping from integers to truth values, and an array xs, returns the number of elements in xs satisfying the predicate.<br>
ZIPWITH :: (int -> int -> int) -> [int] -> [int] -> [int]<br>
Given a lambda function f mapping integer pairs to integers, and two arrays xs and ys,returns the array resulting from applying f to corresponding elements of xs and ys. The length of the returned array is the minimum of the lengths of xs and ys.<br>
SCANL1 :: (int -> int -> int) -> [int] -> [int]<br>
Given a lambda function f mapping integer pairs to integers, and an array xs, returns an array ys of the same length as xs and with its content defined by the recurrence <br>

项目中用到的数据变量集合：<br>
INT-INT  ：(+1), (-1), (*2), (/2), (*(-1)), (**2), (*3), (/3), (*4),(/4)<br>
INT-BOOL ： (>0), (<0), (%2==0), (%2==1)<br>
INT-INT-INT ：  (+), (-), (*), MIN, MAX<br>

综合以上15个指令和19个变量，程序的属性总共有34种，即枚举的空间即这15种指令和19个数据变量的自由组合。可以看出即时不使用神经网络技术来缩减状态空间，只通过暴力枚举依然可以枚举出符合任务描述的程序。<br>

此处特别说明，虽然最终得到的程序语句都是顺序执行的，没有分支和循环结构，但是实际上指令集中包含的指令有些是有分支和循环结构的，比如map指令，sort指令等等。<br>

下面介绍模型的工作流程：<br>
 

 我们需要用到的数据集是一个三元组{P,A,M}
 P代表程序，A代表属性，M代表输入输出示例<br>
 
 我们项目的目的是通过input-output集合M输入给模型，深度学习模块能够预测出程序的属性集合A,然后生成模块根据属性A缩减枚举的状态空间，最终生成代码片段P。<br>
 
 在深度学习模块的训练阶段，我们把input-output集合M作为网络的输入，把属性集合A 通过one-hot成一个1*34维的向量，每个位置上的数值为1代表程序中将包含这条指令或操作数，为0代表不包含。<br>
 
 如：1 1 1 1 0 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0<br>

深度学习模块的结构如下：
![image](https://github.com/ztz818/Auto-programming/blob/master/1.jpg)

神经网络将input-output对作为网络的输入，先分别embedding，然后通过三层的全连接网络获取数字特征，最后对各个输入的向量加和取平均值得到网络的整体输入特征，对特征做一次sigmoid激活，将得到的结果与预测的属性值做回归，然后通过反向传播调整网络的参数，直到预测结果符合要求。实际上神经网络做了一个input-output到Attribute的多标签分类任务。<br>
 （因为此处只用到了数字作为输入，所以rnn,lstm等网络单元实际上并不需要，实验结果表明使用全连接层能获得很好的结果）<br>
 
 最后将深度学习模块得到的预测属性输入给枚举生成模块来缩减状态空间，最终生成符合任务描述的可执行代码。<br>
 
 下图是实验测试500个测试用例训练网络后结果的混淆矩阵。可以看到程序的属性得到很好的预测。
 ![image](https://github.com/ztz818/Auto-programming/blob/master/3.jpg)

 
 
