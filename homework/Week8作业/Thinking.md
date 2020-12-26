# MarTech Challenge 用户购买预测

[TOC]

大赛网址：https://aistudio.baidu.com/aistudio/competition/detail/51



## 特征分析与处理

* **原始数据字段**

  * order_detail_id	订单详情id
    * 理解为针对每种商品的子订单

  * order_id	订单id
    * 理解为一次给到一个商家的订单

  * order_total_num	订单商品总购买数量

    * [x] 等同于同一个订单里子订单购买商品数量order_detail_good_num之和
    * [x] last, mean, sum

  * order_amount	订单商品总金额

    * [x] mean, sum, std

  * order_total_payment	订单实付金额

    * [x] last, mean, std, sum(Monetary)

  * order_total_discount	订单优惠金额

    * [x] mean, sum, std

  * order_pay_time	付款时间

    * [x] 提取月、日、星期几以及小时数据
    * [x] 距离待预测时间过了多少小时

  * order_status	订单状态 

    * 1表示等待买家付款， 2表示卖家部分发货， 3表示卖家发货， 4表示等待买家确认收货， 5表示买家已签收， 6表示交易成功
    * [x] **101可能表示交易失败，也可能表示进行了售后处理**
    * [x] 不知有什么用，暂时不放到数据里

  * order_count	订单包含的子订单数量

    * [x] 不直接放到特征数据里

  * is_customer_rate	用户是否评价，0没有评价，1已经评价

    * [x] 计算用户的评价率

  * order_detail_status	订单详细状态

    * [x] **101可能表示交易失败，也可能表示进行了售后处理**
    * [x] 不知有什么用，暂时不放到数据里

  * order_detail_good_num	订单中的商品数量

    * [x] order_total_num感觉已经够了，暂时不放

  * order_detail_amount	子订单应付总金额

    * [x] 感觉这个特征数据有问题，我更倾向于使用order_detail_payment+order_detail_discount作为子订单应付金额，用来计算子订单优惠率
    * [ ] 不直接用order_detail_payment+order_detail_discount，如果order_amount明显有效再考虑使用

  * order_detail_payment	子订单实付金额

    * 无论order_detail_status是否为101，order_detail_payment都有可能为0

    * [ ] 进一步深挖0元、1元、1角、1分购等商品情况
    * [x] mean, sum, std

  * order_detail_discount	订单优惠金额

    * [x] mean, sum, std

  * member_id	会员id

    * [x] 暂不放入数据中

  * [x] customer_id	用户id

  * customer_gender	性别：0未知，1男，2女

    * [ ] 考虑对性别进行聚类、预测
    * [x] 0值填充

  * customer_province	用户省份所在地

    * [x] last、count（少量有多个）
    * [x] 标签编码
    * 缺失比例0.0494%
      * [x] 填充“未知”
      * [ ] 填充“众数”
      * [ ] 考虑是否删除对应样本数据

  * customer_city	用户城市所在地

    * [x] last、count（少量有多个）
    * [x] 标签编码
    * 缺失比例0.0494%
      * [x] 填充“未知”
      * [ ] 填充“众数”
      * [ ] 考虑是否删除对应样本数据

  * member_status	会员状态：1正常，2冻结，3已删除

    * 说明
      * 取值只有1和缺失值
    * [x] 填充0
    * [x] last

  * is_member_active	会员是否激活：0没有激活，1已激活

    * 说明
      * 取值只有1和缺失值
    * [x] 删除

  * goods_id	商品id

    * 说明
      * 一个order_detail_id同时只对应一个goods_id
      * 一个order_id同时可能包含多个相同的goods_id（所以goods_id可能表示商品类别，而不是单一商品）
    * [x] 标签编码
    * [x] last
    * [ ] 考虑word2vec

  * goods_class_id	商品分类id

    * [x] 与goods_id完全相同，因此删除该列

  * goods_price	商品原始价格

    * 说明
      * 同一个good_id不同子订单的goods_price都有点小差别，猜测可能是脱敏的时候做了一个加减小幅随机值的操作
      * 商品价格goods_price为负值的，对应的实付单价都是1元、1角、1分或者免费的，估计是搞活动引起的
    * 缺失比例
      * [x] 根据good_id进行填充
      * [ ] 直接删除样本处理

    * [x] 暂不使用

  * goods_status	商品库存状态：1出售中，2库中，0未知

    * 一个goods_id只对应一种goods_status
    * [x] 感觉用处不大，暂不使用

  * goods_has_discount	是否支持会员折扣：0不支持，1支持

    * 一个goods_id只对应一种goods_has_discount

    * goods_has_discount为0的商品有796个，为1的有241个

    * goods_has_discount=0时，会员与非会员购买比值为0.4023；goods_has_discount=1时，会员与非会员购买比值为0.1844，显然没有提升，说明

      `train.groupby(["goods_has_discount", "member_status"])["member_status"].count()`

    * [x] 使用占比

  * goods_list_time	商品最新上架时间

    * 说明
      * [ ] 一个goods_id只对应一个goods_list_time
    * [x] diff

  * goods_delist_time	商品最新下架时间

    * 说明
      * 一个goods_id只对应一个goods_delist_time
      * 存在order_pay_time > goods_delist_time的（4227个样本），不太合理
    * [ ] 如果最新下架时间再2013年9月之前，则购买该商品的概率为0
    * [x] diff




* **衍生特征**
  * order_all_discount_rate
  * order_discount_rate
  * order_detail_discount_rate



* **订单优惠关系式**

  * 订单实付金额 = 订单商品总金额 - SUM(各子订单优惠金额) - 商家订单优惠金额

    ​                         =            SUM(各子订单实付金额)                    - 商家订单优惠金额

    * 订单商品总金额 = 订单实付金额 + SUM(各子订单优惠金额) + 订单优惠金额
      $$
      order\_amount = order\_total\_payment + SUM(order\_detail\_discount) + order\_total\_discount
      $$

      * 即使**order_status=101或order_detail_status=101**，上式也成立

    * 订单实付金额 = SUM(各子订单实付金额) - 订单优惠金额
      $$
      order\_total\_payment = SUM(order\_detail\_payment) - order\_total\_discount
      $$

  * 当order_count=1时，子订单实付金额 = 子订单应付金额 - 商家订单优惠金额



## 注意避免“穿越事件”的发生

* 假如用户在7月才在平台产生第一笔交易，那么在6月以前各种数据都是空或0，如果拿这个作为训练数据训练模型会导致结果不好
  * 训练模型只拿之前已有购买行为的数据
* 训练集时间与验证集时间存在交集，出现未卜先知的情况



## 利用滑动时间窗口计算统计特征

* 注意冷启动问题
  * 用户画像+产品画像（区分新老用户，解决冷启动问题，注意用户画像随时间发生的变化）
* 时间范围选择（7天、14天、21天、1个月、3个月等）
* 更接近当前的一些数值赋予更高的权重



## 其他

* 不均衡数据集
  * 过采样与欠采样

* 离散特征Embedding
  * 折中方案，可以考虑其他离散特征处理方法



### 用户画像

* **member_id**
  * member_id = 0
    * 此时customer_gender、member_status和is_member_actived同时为NaN，**很可能是游客**
    * 但customer_gender、member_status和is_member_actived同时为NaN的，有99.998% member_id =0
    * **处理方式：令member_status为缺失值的行对应的member_id = 0**
  * member_id != 0
    * 每个member_id对应一个customer_id
    * **处理方式：member_status填充缺失值为0，用member_status表示member_id 是否不为0，删除is_member_actived**
* 如果在滑动窗口内均未购买，则直接令预测为0
* 对性别进行聚类预测，且取两列性别特征，一列保持原状，另一列则新增预测的性别结果
* 省份和城市
  * 是否考虑按购买比例进行排序再labelencoder？



### 商品画像

* 如果商品下架了，应该直接令预测为0
* 0元、1元、1角、1分购商品