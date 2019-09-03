<!--<center>-->
## 数据标准匹配代码简介
<!--</center>-->
作者： Guobin Li &emsp; &emsp; 日期：2019年8月30日
###1. 运行方法
* http调用python方法：文件位于

	`./fd.core/fd/ng/netserver/fd_http_sever.py`

	在启动python后台之后，就可以与前端进行数据传输了。这里有两个地址

	```
	标准输入地址：后台地址/stdin
	标准匹配地址：后台地址/standard-matching
	```

* rpc调用python方法：文件位于

	`./main/app.py`

	直接启动即可。

* 本地调试：文件位于
	`./main/final_step.py`
	
	直接运行即可。

	<font face="黑体" color=purple size=3>注意：由于两种不同的调用方式它们的文件深度不一样，以下展示了在运行不同的方法调用时需要更改代码的地方。</font>

1. `./main/final_step.py` 文件中的Line49: 

	```
	在http调用情形下为
	standard_dir = os.path.abspath(os.path.join(os.getcwd(), "../../../../resources/standard"))
	而在rpc调用或本地调试情形下为
	standard_dir = os.path.abspath(os.path.join(os.getcwd(), "../resources/standard"))
	```

2. `./main/standard_step.py` 文件中的Line43: 

	```
	在http调用情形下为
	standard_dir = os.path.abspath(os.path.join(os.getcwd(), "../../../../resources/standard"))
	而在rpc调用或本地调试情形下为
	standard_dir = os.path.abspath(os.path.join(os.getcwd(), "../resources/standard"))
	```


3. `./libs/calc_Chinese_similarity.py` 文件中的Line124: 

	```
	在http调用情形下为
	stopping_words_dir = os.path.abspath(os.path.join(os.getcwd(), "../../../../resources/停词表/百度停用词表.txt"))
	而在rpc调用或本地调试情形下为
	stopping_words_dir = os.path.abspath(os.path.join(os.getcwd(), "../resources/停词表/百度停用词表.txt"))
	```


###2. 代码简介
<!--###2.0 目录结构
<center> 表1 数据标准匹配的代码结构

</center>-->
####2.1 `./main`目录

* `./main/app.py`
	
	主要功能：实现rpc调用
	
	主要方法：
	
	```
	get_standard(json_std_str):
	# 此方法用来进行标准的输入与保存。
	# 参数：标准的json数组
	# 返回值：[{"code":200,"message":"ok","data":null}]
	
	get_answer(json_test_str, sim_number=4):
	# 此方法用来进行标注的匹配。
	# 参数：需要匹配的标准的json数组；相似度最高的个数sim_number
	# 返回值：完成匹配标准的json数组
	```

* `./main/final_step.py`

	主要功能：进行个成分相似度的加权累加和与数据标准匹配（利用加权相似度的排名)  
	
	这个文件构造了一个名为FinalStep的类。
	<center> 表2 FinalStep类 
	
	方法名称 | 主要功能 | 备注
	:-- | :-- | :--
	\__init__ |  | 
	final | 加权相似度计算 & 数据标准匹配 | 主算法的最后一步<br>本地调试、rpc调用、http调用的输出主要方法
	</center><br>
	<font color=purple> * 需要注意的代码与解释 </font>  
	
	```
	Line 131-139
	# 权值有待调整
        cfg = FdConfigSingleton()
        # 读取配置文件
        zh_sim_weight = cfg.config.get('similarity_weights').get('zh_sim_weight')
        en_sim_weight = cfg.config.get('similarity_weights').get('en_sim_weight')
        other_sim_weight = cfg.config.get('similarity_weights').get('other_sim_weight')
        total_sim_matrix = zh_sim_weight * zh_sim_matrix + \
                           en_sim_weight * en_sim_matrix + \
                           other_sim_weight * sum(ot_sim_matrix_list) / len(ot_sim_matrix_list)
	# 这里需要通过修改配置文件并读取配置文件来更改&获取各个成分的权重关系
	# 配置文件路径：./fd.core/fd/ng/core/conf/core_config.yaml                    
	```

* `./main/standard_step.py`  

	主要功能：实现标准数据的输入功能。  
	
	这个文件构造了一个名为StandardStep的类。
	
	<center> 表3 StandardStep类
	
	方法名称 | 主要功能 | 备注
	:-- | :-- | :-- 
	\__init__ |  | 
	get\_std\_array | 将json字符串转换成numpy_array | 
	get\_zh\_std\_docs\_list | 获取中文标准文档列表 | 第一维度为不同的文本，<br>第二维度为同一文本中的词语
	get\_zh\_std\_words\_docs\_info | 获取中文分词列表&<br>中文标准单词文本矩阵&<br>中文标准相似度矩阵 | 
	get\_en\_std\_docs\_list | 获取英文标准文档列表 | 第一维度为不同的文本，<br>第二维度为同一文本中的单词
	get\_en\_std\_words\_docs\_info | 获取英文标准分词列表&<br>英文标准单词文本矩阵 | 
	get\_std | 初始化总步骤 | 
	</center><br>
	<!--<font color=purple> * 需要注意的代码与解释 </font>  -->
	
* `./main/user_interface.py`

	主要功能：python本地的用户界面（可忽略）。

####2.2 `./utils`目录

* `./utils/calc_accuracy.py`

	主要功能：计算准确度。
	
	这个文件构造了一个AccuracyCalculator的类。这个类仅有一个可以调用的方法。
	
	```
	calc_accuracy(self, group: list) -> list:
	通过输入一个由元组组成的二维列表，输出准确度列表
	```

* `./utils/list_list_operator.py`

	主要功能：实现列表与嵌套列表的一些操作。
	
	这个文件构造了一个名为ListListOperator的类。
	
	<center> 表4 ListListOperator类
	
	方法名称 | 主要功能 | 备注
	:-- | :-- | :-- 
	\__init__ |  | 
	delete\_repeat | 去掉列表中的重复元素 | 不改变相对顺序（稳定的） 
	calc\_two\_vectors\_sim | 计算两个向量之间的相似度 | 余弦相似度
	list\_number\_contain\_word | 计算包含输入单词的文本数量 | 
	get\_vector\_from\_list| 获取嵌套列表的某一维度上的所有值 | 	get\_column | 获取二维嵌套列表的一列 | 
	dim\_decrease | n维嵌套列表降维 | 
	get\_mid\_from\_list | 三维嵌套列表取出中间一维的值 |
	calc\_two\_documents\_similarity | 计算两文本的相似度 | 全模式
	calc\_two\_documents\_similarity\_naive | 计算两文本相似度 | 朴素模式
	calc\_two\_documents\_similarity\_simple\_naive | 计算两文本相似度 | 简单朴素模式
	cache\_operate | 格式转换 |
	</center><br>

* `./utils/most_similarity_finder.py`
	
	主要功能：寻找最相似的数据标准。
	
	这个文件构造了一个名为MostSimilarityFinder的类。
	
	<center> 表5 MostSimilarityFinder类

	方法名称 | 主要功能 | 备注
	:-- | :-- | :-- 
	\__init__ |  | 
	find\_kth\_largest\_num | 查找最相似的k个标准数据 |  
	find\_the\_most\_similar | 输出最相似标准数据的详细信息 | 
	</center><br>
	
	<font color=orange> * 可以优化的地方 </font>  
	
	```
	def find_kth_largest_num(self, array: list) -> list:
	这个函数在查找前k个最相似的数据标准的时候实际上可以用堆排序，这样子时间复杂度还可以下降。                  
	```

* `./utils/phrase_operator.py`
	
	主要功能：英文字段分词
	
	这个文件构造了一个名为PhraseOperator的类。
	
	<center> 表6 PhraseOperator类

	方法名称 | 主要功能 | 备注
	:-- | :-- | :-- 
	\__init__ |  | 
	is\_letter | 获得当前字母的类别 |  
	get\_next_\begin\_letter | 输出下一个单词的开头的索引 | 
	get\_next\_end\_letter | 输出末尾字母位置 | 
	phrase\_operate | 英文字符串分词操作 | 
	words\_cutter | 将一个中文拼音首字母组成的单词用n_gram拆分 | 针对标准中的中文名称字段为拼音的情形
	</center><br>
	
* `./utils/time_func.py`

	主要功能：计时器。
	
	这个文件只有一个方法，实现了程序的计时功能。
	
	```
	def func_timer(function):
	# 用装饰器实现函数计时
	```

#### 2.3 `./libs` 目录

* `./libs/calc_Chinese_similarity.py`

	主要功能：计算中文字段的相似度。
	
	这个文件构造了一个名为ChineseSimilarity的类，该类继承于以下这些类：WordsDocumentsMatrix, NoneNegtiveFactorization, LatentDA, SingularVectorDecomposition, WordsDictionarySimilarityMatrix.
	
	<center> 表7 ChineseSimilarity类

	方法名称 | 主要功能 | 备注
	:-- | :-- | :-- 
	\__init__ |  | 
	cut\_zh\_word | 进行中文分词 |  
	get\_zh\_words\_sim\_matrix | 获得中文词语的相似度矩阵 | 
	calc\_zh\_sim\_w2v | 计算中文文本间的相似度 | 支持三种模式：0为全模式， 1为朴素模式， 2为简单朴素模式
	phrase\_operate | 英文字符串分词操作 | 
	words\_cutter | 将一个中文拼音首字母组成的单词用n_gram拆分 | 针对标准中的中文名称字段为拼音的情形
	</center><br>
	<font color=purple> * 需要注意的代码与解释 </font> 
	
	```
	# Line21-29
	def __init__(self
                 , list_test=[]
                 , zh_percentage=0.5
                 , consider_word_length=None
                 , zh_std_docs_list=[]
                 , zh_std_split_words_list=[]
                 , zh_std_words_docs_matrix=[]
                 , zh_std_words_sim_matrix=[]
                 , initial=False):
   # 这里的intial表示是否为数据标准输入的步骤
	```
	
* `./libs/calc_English_similarity.py`
	
	主要功能：计算英文字段的相似度。
	
	这个文件构造了一个名为EnglishSimilarity的类，该类继承于以下这些类：WordsDocumentsMatrix, PhraseOperator, NoneNegtiveFactorization, LatentDA, SingularVectorDecomposition.
	
	<center> 表7 ChineseSimilarity类

	方法名称 | 主要功能 | 备注
	:-- | :-- | :-- 
	\__init__ |  | 
	cut\_en\_words | 进行英文分词 |  
	calc\_en\_sim\_svd | 用SVD分解进行英文主题提取 | 
	calc\_en\_sim\_normal | 计算英文文本间的相似度 | 
	get\_en\_words\_documents\_matrix | 获得英文单词-文本矩阵 | 
	</center><br>
	<font color=purple> * 需要注意的代码与解释 </font> 
	
	```
	# Line16-22
	def __init__(self
                 , list_test=[]
                 , en_percentage=0.5
                 , en_std_docs_list=[]
                 , en_std_split_words_list=[]
                 , en_std_words_docs_matrix=[]
                 , initial=False):
   # 这里的intial表示是否为数据标准输入的步骤
	```

* `.utils/calc_other_similarity.py`

	主要功能：计算除中英文之外其他字段的相似度。
	
	这个文件构造了一个名为OtherSimilarity的类。
	
	<center> 表8 ChineseSimilarity类

	方法名称 | 主要功能 | 备注
	:-- | :-- | :-- 
	\__init__ |  | 
	find\_difference | 检查两个字符串是否相同 |  
	clac\_other\_similarity | 计算其它字段的相似度 | 
	</center><br>

* `.utils/words_dic_sim_matrix.py`

	主要功能：生成单词之间的相似度矩阵。
	
	这个文件构造了一个名为WordsDictionarySimilarityMatrix的类，该类继承于ListListOperator类。
	
	<center> 表9 WordsDictionarySimilarityMatrix类

	方法名称 | 主要功能 | 备注
	:-- | :-- | :-- 
	\__init__ |  | 
	generate\_words\_number\_dictionary | 建立单词-序号之间的哈希关系 | 输出{单词: 序号}字典  
	generate\_words\_vector\_dictionary | 建立单词序号-词向量之间的哈希关系 | 输出{单词序号: 词向量}字典
	generate\_words\_similarity\_matrix | 生成单词相似度矩阵 | 标准输入阶段
	generate\_new\_words\_similarity\_matrix | 生成新的单词相似度矩阵 | 匹配标准阶段
	</center><br>
	
* `utils/words_documents_matrix.py`

	主要功能：生成单词-文本矩阵
	
	这个文件构造了一个名为WordsDocumentsMatrix的类，该类继承于ListListOperator类。
	
	<center> 表10 ordsDocumentsMatrix类

	方法名称 | 主要功能 | 备注
	:-- | :-- | :-- 
	\__init__ |  | 
	generate\_stopping\_words\_list | 生成停词列表 |  
	construct\_word\_document\_matrix | 构造单词-文本矩阵 | 标准输入阶段
	tf\_idf | 计算TF-IDF的值 |  
	construct_new_word_document_matrix | 构造新的单词-文本矩阵 | 匹配标准阶段
	</center><br>

	<font color=orange> * 可以优化的地方 </font>  
	
	```
	# Line124-129
	def tf_idf(self
               , words_documents_matrix_temp
               , smooth=True
               , consider_word_length=False
               , flat_words_list=None
               , words_list=None):
   	# 这个计算tf-idf函数并没有调用机器学习包，所以会影响运行速度，若改用现成的包，也许速度会提高。                  
	```
