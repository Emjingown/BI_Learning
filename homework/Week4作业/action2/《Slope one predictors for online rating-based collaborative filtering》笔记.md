## 《Slope one predictors for online rating-based collaborative filtering》笔记

### 1. 算法优势：

1. 易于实现与维护
2. 高效查询
3. 比较准确
4. 支持在线查询与动态更新



### 2. 文章主要贡献

* 提出的Slope one算法的**预测精度**与其他基于内存的预测方法（Pre User Average、Bias from Mean、adjusted cosine item-based、PEARSON scheme）**相近**，但**更能适应协同过滤任务**。



### 3. SlopeOne算法

* 只有已经与被预测用户一起评价了一些共同项目的用户的那些评价，以及只有被预测用户也评价了的那些项目的评价，才进入了坡度一方案下的评价预测。
  $$
  \operatorname{dev}_{j, i}=\sum_{u \in S_{j, i}(\chi)} \frac{u_{j}-u_{i}}{\operatorname{card}\left(S_{j, i}(\chi)\right)}
  $$

  $$
  P(u)_{j}=\frac{1}{\operatorname{card}\left(R_{j}\right)} \sum_{i \in R_{j}}\left(\mathrm{dev}_{j, i}+u_{i}\right)
  $$

  * 当数据集足够稠密时，可近似简化为以下公式：
    $$
    P^{S1}(u)_{j}=\bar{u} + \frac{1}{\operatorname{card}\left(R_{j}\right)} \sum_{i \in R_{j}} \mathrm{dev}_{j, i}
    $$

    * 此时不需要依赖被预测用户对项j的偏好
    * 只依赖于用户的平均评分以及该用户已点评项目的评分

* **缺点**

  * 未考虑评分数量对评分可信度的影响



### 4. 加权SlopeOne算法

* 按训练集中存在成对评分数量进行加权，从而反映评分数量对评分可信度及结果的影响

* 使得评分数量多的项目的评分偏差对预测结果的影响更大
  $$
  P^{w S 1}(u)_{j}=\frac{\sum_{i \in S(u)-\{j\}}\left(\operatorname{dev}_{j, i}+u_{i}\right) c_{j, i}}{\sum_{i \in S(u)-\{j\}} c_{j, i}}
  $$
  其中$c_{j,i}=card(S_{j,i}(\chi))$



### 5. Bi-Polar SlopeOne算法

* 基于每个用户对已点评项目的评分均值，将已点评项目划分为**喜欢的项目集**和**不喜欢的项目集**

* 当利用SlopeOne算法计算偏差时，只考虑对已点评项目具有相同喜好（喜欢或不喜欢）的用户来计算评分偏差
  $$
  \operatorname{dev}_{j, i}^{l i k e}=\sum_{u \in S_{j, i}^{l i k e}(\chi)} \frac{u_{j}-u_{i}}{\operatorname{card}\left(S_{j, i}^{l i k e}(\chi)\right)}
  $$
  <img src="《Slope one predictors for online rating-based collaborative filtering》笔记.assets/image-20201124232545795.png" alt="image-20201124232545795" style="zoom:67%;" />

* 缺点

  * 只利用具有相同喜好的数据来进行模型训练与预测，必然会减少可利用进行模型训练的样本数量
  * 当数据集不够大时，可能会因此而导致欠拟合的现象，模型预测效果不佳

* 优点

  * 考虑相同喜好的用户评分来进行预测更具有可解释性及更符合现实

  * 当数据集足够大时，该算法可能具备更好的泛化能力

    

### 6. 总结

* 从结果来看，SlopeOne算法在EachMovie数据集上表现均优于加权SlopeOne和Bi-Polar SlopeOne算法，在Movielens上表现相同
* 同时这三个算法在预测精确度上基本与其他基于内存的CF算法处于一个水平，但可能更易于实现、更高效
* 从算法原理来看，感觉Bi-Polar SlopeOne算法的可解释性更强，按理说预测效果和泛化能力也会更好；但从算例结果来看反而是表现最差的一个，可能的原因是数据集数据量不够大，或数据本身分布情况对结果的影响
* 因此，当选择推荐算法时，感觉还是需要通过分别测试来找到适用性更好的算法。