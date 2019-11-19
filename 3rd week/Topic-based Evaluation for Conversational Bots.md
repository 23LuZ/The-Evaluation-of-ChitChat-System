## Topic-based Evaluation for Conversational Bots 
## 摘要
- 基于主题的评价指标（topic-based metrics）,评价一个闲聊机器人是否能够在一个主题上保持连贯而有趣的对话，以及是否可以处理多样的主题
- 检测每一话语的主题，使用Deep Average Networks(DAN)训练一个主题分类器
- 通过添加一个基于主题的注意力表（topic-word attention table）对DAN进行了扩展，使得模型不仅能够捕获话语中的关键词还能够进行主题分类
- 实验结果表明，提出的评价方法与人工评价方法具有相关性并且是人工评价方法的补充
- 提出了对话级别的可解释评估指标，用于评估主题的连贯性和多样性
- real conversation-level ratings

## 简介
- 对话可以看作是在一系列主题上信息和观点的交换，因此可以使用基于主题的评价指标来评价闲聊机器人的质量
- 主题广度（topic breadth）：闲聊机器人在不重复表达的情况下就各种粗粒度和细粒度主题进行交谈的能力
- 主题深度（topic depth）：闲聊机器人在给定主题上保持长时间和连贯的对话的能力
- 为了对对话主题进行分类，采用深度平均网络（DAN），并在内部Question数据和Alexa知识查询数据上训练有监督的主题分类器，以根据话语识别主题
- 通过添加一个基于主题的注意力表（topic-word attention table）对DAN进行了扩展，该表可学习词汇表中各个主题词的权重，从而可以检测出每一句话语中的特定主题的关键词
- 这使得主题分类器不仅可以识别诸如“政治”之类的话语的粗粒度主题，而且还可以突出显示诸如“特朗普”之类的细粒度主题词
- 两种模型都用于分析在Alexa奖竞赛中收集的数以万计的人机对话
- 主题分类器和主题关键字检测器在测试集上可以达到良好的准确性，并且可以检测每个主题有意义的关键字
- 机器人保持长时间的特定于主题的连贯子会话的能力与实时用户评分所衡量的用户满意度密切相关
- 某些主题指标（例如主题特定的关键字覆盖率）可以补充人为的判断并发现由于实时数据收集的固有局限性而被用户评分所掩盖的对话系统问题

## 数据集
- 对话数据分析
    - 对话数据  
    用户和闲聊机器人之间成千上万的对话数据，每个对话都有实时用户评分。每个对话的评价轮数为12

    - 实时用户评分  
    在每次对话结束时，系统会提示用户根据是否愿意再次与该社交机器人聊天，以1到5星的等级给该社交机器人评分。
    虽然各个评分都是主观的，但通过在闲聊机器人级别对多个评分求平均，使得平均评分能表明用户满意度并反映闲聊机器人的质量

    - 回复错误率来评价连贯性  
    为了捕获对话的连贯性，我们对成千上万的随机选择的对话进行了标注， 
    每个社交机器人的回复错误率（RER）定义为错误回复（不正确，不相关，不适当）的轮数与总轮数之比
- 训练数据  
我们使用内部的Question数据和Alexa知识查询数据来训练受监督的主题分类模型

Phatic表示非话题性闲聊，包括用户偏好（“我喜欢你”），确认（“谢谢”，“听起来不错”），个人问题（“你好吗”）和其他非话题性讲话

## 主题分类
- DAN 主题分类模型  
DAN是一个词袋神经模型，该模型将每个输入话语中的词向量平均值作为话语表示，话语表示输入全连接层后进入softmax层进行分类。
- 基于注意的DAN：主题关键字检测的可解释模型  
ADAN继续使用词袋模型，根据学习到的注意力权重，对输入词向量进行加权平均获得最后的话语表示。
主题特定的句子表示形式S是K×D矩阵，其中每一行是主题特定的表示形式sk，D是词向量维度。 S输入一系列完全连接的层，并通过softmax分类。
可以使用任何预先训练的向量（例如GloVe和Word2Vec）来初始化词向量矩阵，而话题词注意力矩阵则是随机初始化的，并且可以从数据中学习。 
在测试期间，可以针对每个测试话语检查学习到的主题词权重，找到给定主题下检测最显着的词。 
具体来说，我们针对主题k检查相应的权重矩阵[w1，k，···，wL，k]，根据权重w1，k的降序对单词进行排名，并在话语中选择信息量最高的单词。
ADAN模型可解释，因为它可以揭示对主题分类决策贡献最大的显着词。

## 基于主题的评价指标
首先定义以下概念：
- 特定主题的轮数T：用户的话语和机器人的回复都属于同一主题，不包括Phatic
- 主题连贯子对话S：定义为一系列连续的特定主题轮数，其中至少包含两个连续轮数，但不包括Phatic轮数。
- 子对话的长度ls：定义为子对话包含的特定主题的轮数。 
- 连续对话的lc：定义为特定主题对话的总轮数

对话主题深度
通过子对话的定义，子对话在特定主题上的轮数越多，对话讨论该主题的深度就越大。 假设一个对话Ci有m个子对话Sij，j = 1，...，m
- 对话级别的平均主题深度：用户与机器人一次对话中所有子对话的平均长度
- 系统级别的平均主题深度：用户与机器人所有对话中所有子对话的平均长度

对话主题广度
对话主题广度评价每个对话中主题的多样性，同时评价粗粒度主题如‘体育’和细粒度主题词如‘足球’的多样性。
- 粗粒度主题广度
    - 对话级别的粗粒度主题广度：Br（Ci）是对话Ci中出现不同主题tk的数量，其中每个主题至少持续了一个子对话。 
    较大的Br（Ci）表示用户与机器人交流了多个主题，并且每个主题持续了至少2轮，这表明对话很大可能是愉快的。

    - 系统级别的粗粒度主题广度:对话级别的粗粒度主题广度的平均值
    - 系统级别的粗粒度主题数量：所有对话中包含主题tk的子对话的数量
    - 系统级别的粗粒度主题频率：将不同主题的子对话的数量进行归一化  
    如果系统级别的粗粒度主题频率的标准差较小或熵很高，那么机器人就具有讨论多种主题的能力 
    
- 特定主题关键字覆盖率  
使用ADAN检测到的每个话语权重最高的两个关键字来计算以下指标：
    - 系统级别主题关键字覆盖率：所有对话中不同关键字的总数
    - 系统级别主题关键字数量：所有对话中的所有关键字的总数
    - 系统级别主题关键字频率
    
    
 ## 结果
 - 评价对话主题深度
 假设对话针对特定主题进行的时间越长，对该主题的探索就越深入，用户就越满意。
 结果表明，主题深度与人工判断的相关性很高为0.707
 
 - 评价对话主题广度
     - 粗粒度主题广度评价
     用户与某个机器人就同一对话中的各个主题进行聊天时，每个对话中的平均主题广度Bravg（B）与用户评级具有合理的相关性，
     因此用户倾向于将该机器人评级为更高
     - 细粒度
 
 
 ## 结论
 - 开发了一种新颖的基于注意力的神经主题模型，即注意力深度平均网络（ADAN），该模型可共同学习主题词注意力表以及分类目标
 - 使用DAN和ADAN的结果以及提出的基于主题的指标，评估了机器人质量的多个方面，
 - 用户满意度与长时间且连贯的主题对话非常相关，而话题广度的指标可能会为用户评分提供补充信息，因为由于实时用户数据的固有局限性，
 很难在用户评分中捕获主题的重复性。 
 - 未来的工作将解决如何合并此类指标以提高对话式机器人的质量。 
 - 还计划研究无监督的主题建模方法，以从大量未标记的人机交互中学习潜在的主题，并开发结合了两个交互方的对话上下文的主题模型。
 
 