title: hadoop-combat
date: 2015-06-25 16:21:40
categories: Study Notes
tags: [Hadoop]
---
# MapReduce
Mapper处理的数据是由InputFormat分解过的数据集，其中InputFormat的作用是将数据集切割成小数据集InputSplits，每一个InputSplit将由一个Mapper负责处理。此外，InputFormat中还提供一个RecordReader的实现，并将一个InputSplit解析成`<key,value>`对提供给了map函数。InputFormat的默认值是TextInputFormat，它针对文本文件，按行将文本切割成InputSplits,并用LineRecordReader将InputSplit解析成`<key,value>`对，key是行在文本中的位置，value是文件中的一行。