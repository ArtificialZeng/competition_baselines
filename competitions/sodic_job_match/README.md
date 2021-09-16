# 人岗精准匹配模型 Baseline 分享——怎样简单使用 bert 进行文本特征表示
线下0.8627，线上0.8692

## 赛道链接
https://www.sodic.com.cn/competitions/900008

## 赛道背景
企业招聘需求日益多元化、精细化，招聘服务的开展难度正面临日益严峻的挑战。本赛题期望选手通过自然语言处理、机器学习等前沿技术手段，建立海量企业招聘岗位画像、个人用户画像，在人才推荐、岗位推荐等方向提供数据智能服务，从而提高企业人才招聘效率。

数据包括求职者基本信息，求职意向，工作经历，专业证书，项目经验，招聘岗位信息和招聘结果信息。

## Baseline
Baseline 没做多少复杂的操作，尽可能将各个表的数据合并到一张表，做了一些简单的数据处理和特征工程，并利用 Bert 对文本数据进行特征提取。

### 简单的数据处理
原始数据中的学历是字符串格式，但自身包含一些排序关系，所以根据常识进行了大小映射。工作年限存在同样的情况，也进行人工规则映射。
```
# 学历
{
    '其它': 0,
    '中专': 1,
    '高中（职高、技校）': 2,
    '大专': 3,
    '大学本科': 4,
    '硕士研究生': 5,
    '博士后': 6
}

# 工作年限
{
    '应届毕业生': 0,
    '0至1年': 1,
    '1至2年': 2,
    '3至5年': 3,
    '5年以上': 4,
}    
```
专业要求字段部分存在多余的字符，比如“【园艺学】”和“园艺学”应该属于同一个东西，为了避免二意，删除字段中的“【”和“】”。


### 简单的基础特征
* 工作经历的数量
* count 特征：求职者投递次数，岗位类别次数
* nunique 特征：招聘岗位的投递记录中有多少个不同的岗位类别，招聘岗位的投递记录中有多少个不同的求职者专业
* 数值统计：一份招聘的投递记录中求职者工作年限的mean，max，min，std
* 招聘岗位的工作地点是否和求职者求职意向的工作地点一致
* 招聘岗位的最低学历要求是否大于求职者的最高学历
  
### 简单的文本特征抽取（我瞎整的）
到 https://huggingface.co/nghuyong/ernie-1.0 下载预训练模型 ernie，当然你也可以下载其他预训练模型。将config.json，pytorch_model.bin，vocab.txt放到同一文件夹下（如 baseline 的data/pretrain_models/ernie）。 将文本特征视为句子输入到 ernie 模型，ernie 模型的输出是一个 embedding 序列，baseline 中直接取 cls 对应的 embedding 作为整个句子的表示。baseline 的 sent_to_vec 函数还留有其他的句子表示方式，也可以自己试试。

baseline 中只针对招聘职位，应聘者专业和岗位要求专业这三个文本字段进行了特征抽取，然后利用余弦相似度计算应聘者专业和岗位要求专业的相似度。

原始模型输出的 embedding 维度是非常大的，比如 ernie 的输出维度是768维，要是直接把这样的 embedding 作为特征丢给模型，没加几类文本特征内存就爆炸了。此外在完全无监督的情况下，直接使用预训练 Bert 抽取的句向量，在计算余弦相似度的时候表现可能不是很好。所以使用 BERT-whitening 进行分布校正，具体原理见：https://kexue.fm/archives/8069。BERT-whitening 也可以进行降维操作，减轻内存负担。