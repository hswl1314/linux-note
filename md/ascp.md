---
title: "介绍两种下载方法"
---

> **ascp** 注：须conda activate download\
> **prefetch** 可省略激活步骤

## Ascp

-   进入**NCBI**,在**DataSets**栏下搜索文献测序数据，如bulk码为GSE95424;单细胞码为PRJNA906632;
-   将**BioProject**码复制到www.ebi.ac.uk的Enter acsession栏进行搜索;
-   Show Column Selection的fastq\_\*选项全选中;
-   Download ALL——Generated FASTQ files:FTP,将文件重命名为fq.txt,并上传到服务器。

### ENA下载链接改名

``` python
sed -i 's/wget -nc ftp:\/\/ftp.sra.ebi.ac.uk/fasp.sra.ebi.ac.uk:/g' fq.txt
```

### 运行代码为 nohup ascp.sh \>ascp.log 2\>&1 &

> 下载下来的压缩包一般为\*.fastq.gz

``` python
#!/bin/bash
#ascp.sh 内容如下
#批量下载
cat fq.txt |while read id 
do 
ascp -QT -l 300m -P33001 -k 1 -i ~/.conda/envs/download/etc/asperaweb_id_dsa.openssh era-fasp@$id .
done

#单独下载
ascp -QT -l 300m -P33001 -k 1 -i ~/.conda/envs/download/etc/asperaweb_id_dsa.openssh era-fasp@将①改名后的内容选择一个复制进来 .
```

### 将文件转移到raw_data

``` python
mkdir raw_data
mv *.fastq.gz raw_data
```

## prefetch

-   进入**NCBI**,在**DataSets**栏下搜索文献测序数据，如bulk码为GSE95424;单细胞码为PRJNA906632
-   点击SRA Run Selector进入网页，再点击select选项下的Accession List下载
-   获得SRR_Acc_List.txt文件，并上传到服务器 注：打开此文件光标要位于空白行的首位。

### 下载SRR文件

``` python
mkdir -p raw_data
#批量下载
nohup prefetch -O ./raw_data --option-file SRR_Acc_List.txt >prefetch.log 2>&1 &

#单独下载一个
prefetch SRRXXXXXXXX
```

### 把当前文件夹的下一级子文件夹中的sra文件移动到当前文件夹来

``` python
find . -mindepth 2 -maxdepth 2 -type f -name "*.sra" -exec mv {} . \;
```

### 将SRA文件转化为fastq文件,完成后删除\*.sra文件

``` python
#-S是--split-files，#-e 10开十线程，#*.sra所有sra文件 --include-technical为单细胞才加
nohup fasterq-dump -S -e 10 *.sra >faster1.log 2>&1 &


#转化单独一个文件
fasterq-dump -S -e 10 SRRXXXXXXX.sra
```

### 将fastq文件压缩为fq.gz文件

``` python
#-k 保留原文件，-d是解压
nohup gzip  *.fastq >gzip.log 2>&1 &

#压缩单独一个
gzip SRRXXXXXX.fastq
```
