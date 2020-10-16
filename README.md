# Qiime2Bugbase
### 1.将gg_13_5(bugbase要求为13-5)的rep_set/99_otus_fasta导入qiime2

```
qiime tools import --input-path ~/gg_13_5_otus/rep_set/99_otus.fasta --output-path 99_otus.qza --type FeatureData[Sequence]
```
### 2.vsearch聚类，dada2降噪后生成的*rep-seqs.qza和table.qza*为输入文件，*gg_otus.qza*为参考文件

```
qiime vsearch cluster-features-closed-reference \
--i-sequences rep-seqs-dada2.qza \
--i-table table-dada2.qza \
--i-reference-sequences gg_99_otus.qza \
--p-perc-identity 0.97 \
--o-clustered-table table-cr-99.qza \
--o-clustered-sequences rep-seqs-cr-99.qza \
--o-unmatched-sequences unmatched-seqs \
--verbose
```

### 3. bugbase输入文件为biom 1.0 JSON格式，且列名为taxonomy，而非OTUid，故先提取otutab再将OTUid转化为taxonomy
```
qiime tools export \
--input-path table-cr-99.qza \
--output-path $PWD
```
#得到文件feature-table.biom，但为#OTU ID，需使用gg_13_5/taxonomy/99_otu_taxonomy.txt进行转换

### 4.为99_otu_taxonomy.txt添加列名(#OTUID，taxonomy）
```
echo -e "#OTUID\ttaxonomy" | cat - 99_otu_taxonomy.txt > 99_otu_taxonomy.txt
```

### 5.将taxonomy列添加到biom文件中
```
biom add-metadata -i feature-table.biom -o feature-table-tax.biom --observation-metadata-fp 99_otu_taxonomy.txt --sc-separated taxonomy
```
### 6.将默认的biom2.0文件转为biom1.0（JSON）文件格式：
+ 将biom2.0转为txt格式
```
biom convert --table-type="OTU table" -i feature-table-tax.biom -o feature-table-tax.txt --to-tsv --header-key taxonomy
```
+ 将txt格式转为biom1.0格式
```
biom convert -i feature-table-tax.txt -o feature-table-tax-biom1.biom --table-type="OTU table" --to-json --process-obs-metadata taxonomy
```
### 7.将metadata文件的列名设置为#SampleID
