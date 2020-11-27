# 《Slope one predictors for online rating-based collaborative filtering》笔记



### 1. 算法优势：

1. 易于实现与维护
2. 高效查询
3. 比较准确
4. 支持在线查询与动态更新



### 2. 文章主要贡献

提出的Slope one算法的预测精度与其他基于内存的预测方法（Pre User Average、Bias from Mean、adjusted cosine item-based、PEARSON scheme）相近，但更能适应协同过滤任务。



只有已经与被预测者用户一起评价了一些共同项目的用户的那些评价，以及只有被预测者用户也评价了的那些项目的评价，才进入了坡度一方案下的评价预测。



* SlopeOne算法

  * 

  * $$
    \operatorname{dev}_{j, i}=\sum_{u \in S_{j, i}(\chi)} \frac{u_{j}-u_{i}}{\operatorname{card}\left(S_{j, i}(\chi)\right)}
    $$

  * $$
    P(u)_{j}=\frac{1}{\operatorname{card}\left(R_{j}\right)} \sum_{i \in R_{j}}\left(\mathrm{dev}_{j, i}+u_{i}\right)
    $$

  * $$
    P^{S1}(u)_{j}=\bar{u} + \frac{1}{\operatorname{card}\left(R_{j}\right)} \sum_{i \in R_{j}} \mathrm{dev}_{j, i} 
    $$

  * 

  * 当数据集足够稠密时，可近似简化为该公式；此时不需要依赖被预测用户对项j的偏好

    * 实现只依赖于用户的平均评分以及该用户已点评项目的评分

  * 缺点：不考虑评分数量，可信度

  

* 加权SlopeOne算法

  * 按训练集中存在成对评分数量进行加权
  
  * $$
    P^{w S 1}(u)_{j}=\frac{\sum_{i \in S(u)-\{j\}}\left(\operatorname{dev}_{j, i}+u_{i}\right) c_{j, i}}{\sum_{i \in S(u)-\{j\}} c_{j, i}}
    $$
  
  * 其中$c_{j,i}=card(S_{j,i}(\chi))$
  
* 

* 



* BI-POLAR SLOPE ONE
  * 首先，在项目方面，只考虑两个喜欢的项目之间的偏差或两个不喜欢的项目之间的偏差。第二，在用户方面，只有对项目I和J都打分并且对项目I有共同喜好或不喜欢的用户对的偏差才被用来预测项目J的评分。
  
  * 两极限制必然会减少预测计算中的总评级数量，尽管在数据稀疏是一个问题的情况下，由于这种削减而导致的准确度的任何提高似乎都有违常理，但未能过滤掉不相关的评级可能会被证明是更有问题的
  
  * 重要的是，双极斜率1方案从用户A喜欢项目K而用户B不喜欢同一项目K的事实中预测不到任何东西。
  
  * 将评估训练集分成喜欢集与不喜欢集
  
  * $$
    \operatorname{dev}_{j, i}^{l i k e}=\sum_{u \in S_{j, i}^{l i k e}(\chi)} \frac{u_{j}-u_{i}}{\operatorname{card}\left(S_{j, i}^{l i k e}(\chi)\right)}
    $$
  
  * <img src="《Slope one predictors for online rating-based collaborative filtering》笔记.assets/image-20201124232545795.png" alt="image-20201124232545795" style="zoom:67%;" />
  
* 

超出边界，取边界



从结果来看，SlopeOne算法在EachMovie数据集上表现均优于加权SlopeOne和Bi-Polar SlopeOne算法，在Movielens上表现相同；同时这三个算法在预测精确度上基本与其他基于内存的CF算法处于一个水平，但可能更易于实现、更高效。

