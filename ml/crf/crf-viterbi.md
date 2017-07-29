# 维特比算法解码

---

# 1. linear-CRF模型参数学习思路

　　　　在linear-CRF模型参数学习问题中，我们给定训练数据集X和对应的标记序列Y，K个特征函数f\_k\(x,y\)，需要学习linear-CRF的模型参数w\_k和条件概率P\_w\(y\|x\)，其中条件概率P\_w\(y\|x\)和模型参数w\_k满足一下关系：P\_w\(y\|x\) = P\(y\|x\) =  \frac{1}{Z\_w\(x\)}exp\sum\limits\_{k=1}^Kw\_kf\_k\(x,y\) =  \frac{exp\sum\limits\_{k=1}^Kw\_kf\_k\(x,y\)}{\sum\limits\_{y}exp\sum\limits\_{k=1}^Kw\_kf\_k\(x,y\)}

　　　　所以我们的目标就是求出所有的模型参数w\_k，这样条件概率P\_w\(y\|x\)可以从上式计算出来。

　　　　求解这个问题有很多思路，比如梯度下降法，牛顿法，拟牛顿法。同时，这个模型中P\_w\(y\|x\)的表达式和[最大熵模型原理小结](http://www.cnblogs.com/pinard/p/6093948.html)中的模型一样，也可以使用最大熵模型中使用的改进的迭代尺度法\(improved iterative scaling, IIS\)来求解。

　　　　下面我们只简要介绍用梯度下降法的求解思路。

# 2. linear-CRF模型参数学习之梯度下降法求解

　　　　在使用梯度下降法求解模型参数之前，我们需要定义我们的优化函数，一般极大化条件分布P\_w\(y\|x\)的对数似然函数如下：L\(w\)=  log\prod\_{x,y}P\_w\(y\|x\)^{\overline{P}\(x,y\)} = \sum\limits\_{x,y}\overline{P}\(x,y\)logP\_w\(y\|x\)

　　　　其中\overline{P}\(x,y\)为经验分布，可以从先验知识和训练集样本中得到,这点和最大熵模型类似。为了使用梯度下降法，我们现在极小化f\(w\) = -L\(P\_w\)如下：\begin{align}f\(w\) & = -\sum\limits\_{x,y}\overline{P}\(x,y\)logP\_w\(y\|x\) \\ &=  \sum\limits\_{x,y}\overline{P}\(x,y\)logZ\_w\(x\) - \sum\limits\_{x,y}\overline{P}\(x,y\)\sum\limits\_{k=1}^Kw\_kf\_k\(x,y\) \\& =  \sum\limits\_{x}\overline{P}\(x\)logZ\_w\(x\) - \sum\limits\_{x,y}\overline{P}\(x,y\)\sum\limits\_{k=1}^Kw\_kf\_k\(x,y\) \\& =  \sum\limits\_{x}\overline{P}\(x\)log\sum\limits\_{y}exp\sum\limits\_{k=1}^Kw\_kf\_k\(x,y\) - \sum\limits\_{x,y}\overline{P}\(x,y\)\sum\limits\_{k=1}^Kw\_kf\_k\(x,y\)  \end{align}

　　　　对w求导可以得到：\frac{\partial f\(w\)}{\partial w} = \sum\limits\_{x,y}\overline{P}\(x\)P\_w\(y\|x\)f\(x,y\) -  \sum\limits\_{x,y}\overline{P}\(x,y\)f\(x,y\)

　　　　有了w的导数表达书，就可以用梯度下降法来迭代求解最优的w了。注意在迭代过程中，每次更新w后，需要同步更新P\_w\(x,y\),以用于下一次迭代的梯度计算。

　　　　梯度下降法的过程这里就不累述了，如果不熟悉梯度下降算法过程建议阅读之前写的[梯度下降（Gradient Descent）小结](http://www.cnblogs.com/pinard/p/5970503.html)。以上就是linear-CRF模型参数学习之梯度下降法求解思路总结。

# 3. linear-CRF模型维特比算法解码思路

　　　　现在我们来看linear-CRF的第三个问题：解码。在这个问题中，给定条件随机场的条件概率P\(y\|x\)和一个观测序列x,要求出满足P\(y\|x\)最大的序列y。

　　　　这个解码算法最常用的还是和HMM解码类似的维特比算法。到目前为止，我已经在三个地方讲到了维特比算法，第一个是[文本挖掘的分词原理](http://www.cnblogs.com/pinard/p/6677078.html)中用于中文分词，第二个是[隐马尔科夫模型HMM（四）维特比算法解码隐藏状态序列](http://www.cnblogs.com/pinard/p/6991852.html)中用于HMM解码。第三个就是这一篇了。

　　　　维特比算法本身是一个动态规划算法，利用了两个局部状态和对应的递推公式，从局部递推到整体，进而得解。对于具体不同的问题，仅仅是这两个局部状态的定义和对应的递推公式不同而已。由于在之前已详述维特比算法，这里就是做一个简略的流程描述。

　　　　对于我们linear-CRF中的维特比算法，我们的第一个局部状态定义为\delta\_i\(l\),表示在位置i标记l各个可能取值\(1,2...m\)对应的非规范化概率的最大值。之所以用非规范化概率是，规范化因子Z\(x\)不影响最大值的比较。根据\delta\_i\(l\)的定义，我们递推在位置i+1标记l的表达式为：\delta\_{i+1}\(l\) = \max\_{1 \leq j \leq m}\{\delta\_i\(j\) + \sum\limits\_{k=1}^Kw\_kf\_k\(y\_{i} =j,y\_{i+1} = l,x,i\)\}\;, l=1,2,...m

　　　　和HMM的维特比算法类似，我们需要用另一个局部状态\Psi\_{i+1}\(l\)来记录使\delta\_{i+1}\(l\)达到最大的位置i的标记取值,这个值用来最终回溯最优解，\Psi\_{i+1}\(l\)的递推表达式为：\Psi\_{i+1}\(l\) = arg\;\max\_{1 \leq j \leq m}\{\delta\_i\(j\) + \sum\limits\_{k=1}^Kw\_kf\_k\(y\_{i} =j,y\_{i+1} = l,x,i\)\}\; ,l=1,2,...m

# 4. linear-CRF模型维特比算法流程

　　　　现在我们总结下 linear-CRF模型维特比算法流程：

　　　　输入：模型的K个特征函数，和对应的K个权重。观测序列x=\(x\_1,x\_2,...x\_n\),可能的标记个数m

　　　　输出：最优标记序列y^\* =\(y\_1^\*,y\_2^\*,...y\_n^\*\)

　　　　1\) 初始化：\delta\_{1}\(l\) = \sum\limits\_{k=1}^Kw\_kf\_k\(y\_{0} =start,y\_{1} = l,x,i\)\}\;, l=1,2,...m\Psi\_{1}\(l\) = start\;, l=1,2,...m

　　　　2\) 对于i=1,2...n-1,进行递推：\delta\_{i+1}\(l\) = \max\_{1 \leq j \leq m}\{\delta\_i\(j\) + \sum\limits\_{k=1}^Kw\_kf\_k\(y\_{i} =j,y\_{i+1} = l,x,i\)\}\;, l=1,2,...m\Psi\_{i+1}\(l\) = arg\;\max\_{1 \leq j \leq m}\{\delta\_i\(j\) + \sum\limits\_{k=1}^Kw\_kf\_k\(y\_{i} =j,y\_{i+1} = l,x,i\)\}\; ,l=1,2,...m

　　　　3\) 终止：y\_n^\* = arg\;\max\_{1 \leq j \leq m}\delta\_n\(j\)

　　　　4\)回溯：y\_i^\* = \Psi\_{i+1}\(y\_{i+1}^\*\)\;, i=n-1,n-2,...1

　　　　最终得到最优标记序列y^\* =\(y\_1^\*,y\_2^\*,...y\_n^\*\)

# 5. linear-CRF模型维特比算法实例

　　　　下面用一个具体的例子来描述 linear-CRF模型维特比算法，例子的模型和CRF系列第一篇中一样，都来源于《统计学习方法》。

　　　　假设输入的都是三个词的句子，即X=\(X\_1,X\_2,X\_3\),输出的词性标记为Y=\(Y\_1,Y\_2,Y\_3\),其中Y \in \{1\(名词\)，2\(动词\)\}

　　　　这里只标记出取值为1的特征函数如下：t\_1 =t\_1\(y\_{i-1} = 1, y\_i =2,x,i\), i =2,3,\;\;\lambda\_1=1

t\_2 =t\_2\(y\_1=1,y\_2=1,x,2\)\;\;\lambda\_2=0.5

t\_3 =t\_3\(y\_2=2,y\_3=1,x,3\)\;\;\lambda\_3=1

t\_4 =t\_4\(y\_1=2,y\_2=1,x,2\)\;\;\lambda\_4=1

t\_5 =t\_5\(y\_2=2,y\_3=2,x,3\)\;\;\lambda\_5=0.2

s\_1 =s\_1\(y\_1=1,x,1\)\;\;\mu\_1 =1

s\_2 =s\_2\( y\_i =2,x,i\), i =1,2,\;\;\mu\_2=0.5

s\_3 =s\_3\( y\_i =1,x,i\), i =2,3,\;\;\mu\_3=0.8

s\_4 =s\_4\(y\_3=2,x,3\)\;\;\mu\_4 =0.5

　　　　求标记\(1,2,2\)的最可能的标记序列。

　　　　首先初始化:\delta\_1\(1\) = \mu\_1s\_1 = 1\;\;\;\delta\_1\(2\) = \mu\_2s\_2 = 0.5\;\;\;\Psi\_{1}\(1\) =\Psi\_{1}\(2\) = start

　　　　接下来开始递推，先看位置2的：

\delta\_2\(1\) = max\{\delta\_1\(1\) + t\_2\lambda\_2+\mu\_3s\_3, \delta\_1\(2\) + t\_4\lambda\_4\} = max\{1+0.5+0.8,0.5+1\} =2.3\;\;\;\Psi\_{2}\(1\) =1

\delta\_2\(2\) = max\{\delta\_1\(1\) + t\_1\lambda\_1+\mu\_2s\_2, \delta\_1\(2\) + \mu\_2s\_2\} = max\{1+0.5+0.8,0.5+0.5\} =2.5\;\;\;\Psi\_{2}\(2\) =1

　　　　再看位置3的：

\delta\_3\(1\) = max\{\delta\_2\(1\) +\mu\_3s\_3, \delta\_2\(2\) + t\_3\lambda\_3+\mu\_3s\_3\} = max\{2.3+0.8,2.5+1+0.8\} =4.3\Psi\_{3}\(1\) =2

\delta\_3\(2\) = max\{\delta\_2\(1\) +t\_1\lambda\_1 + \mu\_4s\_4, \delta\_2\(2\) + t\_5\lambda\_5+\mu\_4s\_4\} = max\{2.3+1+0.5,2.5+0.2+0.5\} =3.8\Psi\_{3}\(2\) =1

　　　　最终得到y\_3^\* =\arg\;max\{\delta\_3\(1\), \delta\_3\(2\)\},递推回去，得到：y\_2^\* = \Psi\_3\(1\) =2\;\;y\_1^\* = \Psi\_2\(2\) =1

　　　　即最终的结果为\(1,2,1\),即标记为\(名词，动词，名词\)。

# 6.linear-CRF vs HMM

　　　　linear-CRF模型和HMM模型有很多相似之处，尤其是其三个典型问题非常类似，除了模型参数学习的问题求解方法不同以外，概率估计问题和解码问题使用的算法思想基本也是相同的。同时，两者都可以用于序列模型，因此都广泛用于自然语言处理的各个方面。

　　　　现在来看看两者的不同点。最大的不同点是linear-CRF模型是判别模型，而HMM是生成模型，即linear-CRF模型要优化求解的是条件概率P\(y\|x\),则HMM要求解的是联合分布P\(x,y\)。第二，linear-CRF是利用最大熵模型的思路去建立条件概率模型，对于观测序列并没有做马尔科夫假设。而HMM是在对观测序列做了马尔科夫假设的前提下建立联合分布的模型。

　　　　最后想说的是，只有linear-CRF模型和HMM模型才是可以比较讨论的。但是linear-CRF是CRF的一个特例，CRF本身是一个可以适用于很复杂条件概率的模型，因此理论上CRF的使用范围要比HMM广泛的多。
