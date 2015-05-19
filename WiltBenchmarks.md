# Introduction #

To give an idea of the performance of Wilt, here are some benchmarks for the [Canterbury Corpus](http://corpus.canterbury.ac.nz/index.html), alongside results for gzip and bzip2. The winner for each file has been highlighted.

## The Canterbury Corpus ##

| |text|fax|Csrc|Excl|SPRC|tech|poem|html|list|man|play|Average|
|:|:---|:--|:---|:---|:---|:---|:---|:---|:---|:--|:---|:------|
|Wilt|2.76|0.88|2.21| **0.86** | **2.47** |2.46|3.02|2.62| **2.61** | **3.28** |3.08|2.39   |
|bzip2 -9| **2.27** | **0.78** | **2.18** |1.01|2.70| **2.02** | **2.42** | **2.48** |2.79|3.33| **2.53** | **2.23** |
|gzip -b|2.85|0.82|2.24|1.63|2.67|2.71|3.23|2.59|2.65|3.31|3.12|2.53   |

## The Artificial Corpus ##

| |a|aaa|alphabet|random|
|:|:|:--|:-------|:-----|
|Wilt| **96.00** | **0.00** | **0.00** |6.33  |
|bzip2 -9|296.00| **0.00** |0.04    | **6.05** |
|gzip -b|168.00|0.01|0.02    | **6.05** |

## The Large Corpus ##

Not tested, because the current compressor is horribly slow.

## The Miscellaneous Corpus ##

|   | pi | Average |
|:--|:---|:--------|
| Wilt | 3.72 | 3.72    |
| bzip2 -9 | **3.45** | **3.45** |
| gzip -b | 3.76 | 3.76    |

## The Calgary Corpus ##

|   | bib | book1 | book2 | geo | news | obj1 | obj2 | paper1 | paper2 | pic | progc | progl | progp | trans | Average |
|:--|:----|:------|:------|:----|:-----|:-----|:-----|:-------|:-------|:----|:------|:------|:------|:------|:--------|
| Wilt | 2.46 | 2.98  | 2.49  | 4.90 | 2.81 | **3.75** | 2.54 | 2.78   | 2.86   | 0.88 | 2.71  | 1.82  | 1.79  | 1.57  | 2.60    |
| bzip2 -9 | **1.97** | **2.42** | **2.06** | **4.45** | **2.52** | 4.01 | **2.48** | **2.49** | **2.44** | **0.78** | **2.53** | **1.74** | **1.74** | **1.53** | **2.37** |
| gzip -b | 2.51 | 3.25  | 2.70  | 5.34 | 3.06 | 3.84 | 2.63 | 2.79   | 2.89   | 0.82 | 2.68  | 1.80  | 1.81  | 1.61  | 2.69    |