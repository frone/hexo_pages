---
title: 图像特征抽取
date: 2016-09-26 14:47:25
tags:
    - opencv
    - bovw
    - imagehash
---
<Excerpt in index | 首页摘要>
pHash, BOV等图像特征抽取方法<!-- more -->
<The rest of contents | 余下全文>


代码库地址： git@192.168.30.251:similarCards.git

### 使用hash函数将图片转化为指纹

参照： https://github.com/paul-schwendenman/imagehash2

使用imagehash获取图片的fingerprint，建立指纹库，然后每次查找指纹的编辑距离最近的几个图片作为候选

### 使用BOVW算法

参见代码：192.168.30.251:/opt/workspace/imgSimilarity/clustering/vocabulary.py

思路：
  - 抽取descriptor：使用opencv模块获取图像的keypoints之后，抽取descriptor，
  - 聚类获取码表：由于descriptor是K维向量(列固定)表示的，因此可以用descriptor的向量聚类出一个codebook，每个向量有一个index，
  - 图片重新表示：每张图片都由这个codebook中的向量表示，即用一组index表示。
  - 相似度计算： 两张图片的相似度就是两组index的相似度。

### 其他方法
  - pHash（名片图片集上效果不理想）
  - LIRE

> 基于Lire的图像检索：
—— by mazf
>1、代码：git@192.168.30.251:OraImageRetrival.git
>2、代码说明：
>> 2.1 代码有两个部分，都是Java文件，有详细的注释：
>>--OraIndexer：用来构建索引
>>--OraSearcher：根据构建好的索引，进行图像检索
>>>**OraIndexer** 是一个单线程程序，所以，构建索引比较慢，解决方法是加入多线程
要创建索引，需要一个文档构建对象LocalDocumentBuilder，并给该对象传入提取图像特征的方法，Lire提供了很多特征提取的方法，本程序采用的是快速sift方法CvSurfExtractor。有了LocalDocumentBuilder对象后，可以对图像创建一文档Document，并将该文档写入索引，代码如下：
</br>*// 创建一个文档对象（Lucene的概念）
LocalDocumentBuilder localDocumentBuilder = new LocalDocumentBuilder();
// 给文档对象添加一个提取器，使用Surf的提取器，传入码表
localDocumentBuilder.addExtractor(CvSurfExtractor.class, Cluster.readClusters(codeBook));
// 读取图像文件，并用文档对象创建一个文档
BufferedImage img = ImageIO.read(new FileInputStream(imageFilePath));
Document doc = localDocumentBuilder.createDocument(img, imageFilePath);
// 将图像文档写入索引，iw是IndexWriter对象
iw.addDocument(doc);*
>>></br>
**OraSearch** 是图像检索对象，检索速度在2s左右，很快。
检索和创建索引要匹配，都需要使用相同的特征提取方法，这里使用CvSurfExtractor特征提取
检索的过程需要三个主要的对象：索引读取对象，搜索对象，匹配对象
</br>*// 创建 索引读取对象，搜索对象，匹配对象
// 同样，需要将CvSurfExtractor传给搜索对象，hits中保存的是匹配命中的图像文档对象
IndexReader reader = DirectoryReader.open(FSDirectory.open(Paths.get(indexPath)));
ImageSearcher searcher = new GenericFastImageSearcher(resCnt, CvSurfExtractor.class, new BOVW(),128, true, reader, codeBook);
ImageSearchHits hits = searcher.search(ImageIO.read(new File(imageName)), reader);*</br>




>>2.2 目录介绍
>>--CardImages：用于测试的名片图片
>>--CodeBook： CvSURF128码表，不需要修改
>>--ImageIndex： Lire生成的索引目录

>3、备注：
>程序在Windows下编译，如果在Linux和Mac上，要重新编译Lire库
>Lire源程序在 251 上我的目录下：Lire-1.0b2.zip
>用idea打开，用 maven 编译就可以了


### 相关调研工作见：(192.168.30.251)
  /opt/workspace/imgSimilarity
  /opt/workspace/similarCards


  [1]: git@192.168.30.251:OraImageRetrival.git