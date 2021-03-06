---
layout: post
title: "Running LDA on mahout"
comments: true
categories: [java, mahout, lda]
tags: [java, nahout, lda]
date: 2012-11-11
published: true
---

*(Jan 28 2014): Found this old draft I never got around to polish. It's very rough, but this is a developer notebook after all, so here it goes out into the world.*  

* Install mahout
* Run as core (Create new file)
* Create sequence files from text in directory

```
bin/mahout seqdirectory -c UTF-8 -i data/documents/2012/ -o seqfiles
```

* Create document vectors

```
bin/mahout seq2sparse -i seqfiles/ -o normalized-3-gram -ow -a org.apache.lucene.analysis.WhitespaceAnalyzer -chunk 200 -s 5 -md 3 -x 90 -ng 3 -ml 50 -n 2 -seq
bin/mahout seq2sparse -i seqfiles/ -o normalized-3-gram -ow -chunk 200 --minSupport 5 --minDF 3 --maxDFPercent 90 -ng 3 -ml 50 -n 2 -seq
```

* Get dictionary size (to pass into cvb)

```
bin/mahout seqdumper -i normalized-3-gram/dictionary.file-0 -c
```

* Create vectors in format for cvb. Creates two files. ```matrix``` has sparse vector and ```docIndex``` has mapping from numeric key to original key

```
bin/mahout rowid -i normalized-3-gram/tf-vectors -o normalized-3-gram/tf-vectors-cvb
```

* run cvb

```
bin/mahout cvb  -i normalized-3-gram/tf-vectors-cvb/matrix -o normalized-3-gram/cvb -k 100 -ow -x 20 -nt 640 -dict normalized-3-gram/dictionary.file-0 -dt normalized-3-gram/cvb-doc-topics-10
```

* View index

```
bin/mahout seqdumper -i normalized-3-gram/tf-vectors-cvb/docIndex
```

* view results

```
bin/mahout vectordump -i normalized-3-gram/cvb-10 -o prob3 -d normalized-3-gram/dictionary.file-0 -dt sequencefile -p 1 -sort true -vs 5
```

* dump docs related to topics

```
bin/mahout vectordump -i normalized-3-gram/cvb-doc-topics-10/ -o prob5 -p 1 
```

* doc id mappings

```
bin/mahout seqdumper -i normalized-3-gram/tf-vectors-cvb/docIndex -o doc-id-mappings 
```

* JAVA OPTS

```
-Xmx12000m -server -XX:+UseParallelGC -XX:+UseParallelOldGC -Xms8000m
```

## RUN IT
```
nohup /mahout/bin/mahout cvb  -i normalized-3-gram/tf-vectors-cvb/matrix -o normalized-3-gram/cvb-100 -k 100 -ow -nt 237731 -dict normalized-3-gram/dictionary.file-0 -dt normalized-3-gram/cvb-doc-topics-100 -x 20 &
```