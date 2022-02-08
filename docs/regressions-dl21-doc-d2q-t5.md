# Anserini: Regressions for [DL21 (Doc)](https://microsoft.github.io/msmarco/TREC-Deep-Learning.html)

This page describes experiments, integrated into Anserini's regression testing framework, for the TREC 2021 Deep Learning Track (Document Ranking Task) on the MS MARCO V2 document collection (with doc2query-T5 expansions) using relevance judgments from NIST.

At the time this regression was created (November 2021), the qrels are only available to TREC participants.
You must download the qrels from NIST's "active participants" password-protected site and place at `src/main/resources/topics-and-qrels/qrels.dl21-doc.txt`.
The qrels will be added to Anserini when they are publicly released in Spring 2022.

Note that the NIST relevance judgments provide far more relevant documents per topic, unlike the "sparse" judgments provided by Microsoft (these are sometimes called "dense" judgments to emphasize this contrast).
For additional instructions on working with MS MARCO V2 document collection, refer to [this page](experiments-msmarco-v2.md).

Note that there are four different regression conditions for this task, and this page describes the following:

+ **Indexing Condition:** each document in the MS MARCO V2 document collection is treated as a unit of indexing
+ **Expansion Condition:** doc2query-T5

The exact configurations for these regressions are stored in [this YAML file](../src/main/resources/regression/dl21-doc-d2q-t5.yaml).
Note that this page is automatically generated from [this template](../src/main/resources/docgen/templates/dl21-doc-d2q-t5.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead.

## Indexing

Typical indexing command:

```
target/appassembler/bin/IndexCollection \
  -collection MsMarcoV2DocCollection \
  -input /path/to/msmarco-v2-doc-d2q-t5 \
  -index indexes/lucene-index.msmarco-v2-doc-d2q-t5/ \
  -generator DefaultLuceneDocumentGenerator \
  -threads 24 -storePositions -storeDocvectors -storeRaw \
  >& logs/log.msmarco-v2-doc-d2q-t5 &
```

The value of `-input` should be a directory containing the compressed `jsonl` files that comprise the corpus.
See [this page](experiments-msmarco-v2.md) for additional details.

For additional details, see explanation of [common indexing options](common-indexing-options.md).

## Retrieval

Topics and qrels are stored in [`src/main/resources/topics-and-qrels/`](../src/main/resources/topics-and-qrels/).
The regression experiments here evaluate on the 57 topics for which NIST has provided judgments as part of the TREC 2021 Deep Learning Track.
<!-- The original data can be found [here](https://trec.nist.gov/data/deep2021.html). -->

After indexing has completed, you should be able to perform retrieval as follows:

```
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-v2-doc-d2q-t5/ \
  -topics src/main/resources/topics-and-qrels/topics.dl21.txt -topicreader TsvInt \
  -output runs/run.msmarco-v2-doc-d2q-t5.bm25-default.topics.dl21.txt \
  -hits 1000 -bm25 &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-v2-doc-d2q-t5/ \
  -topics src/main/resources/topics-and-qrels/topics.dl21.txt -topicreader TsvInt \
  -output runs/run.msmarco-v2-doc-d2q-t5.bm25-default+rm3.topics.dl21.txt \
  -hits 1000 -bm25 -rm3 &
```

Evaluation can be performed using `trec_eval`:

```
tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m map src/main/resources/topics-and-qrels/qrels.dl21-doc.txt runs/run.msmarco-v2-doc-d2q-t5.bm25-default.topics.dl21.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 src/main/resources/topics-and-qrels/qrels.dl21-doc.txt runs/run.msmarco-v2-doc-d2q-t5.bm25-default.topics.dl21.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.1000 src/main/resources/topics-and-qrels/qrels.dl21-doc.txt runs/run.msmarco-v2-doc-d2q-t5.bm25-default.topics.dl21.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m recip_rank -c -m ndcg_cut.10 src/main/resources/topics-and-qrels/qrels.dl21-doc.txt runs/run.msmarco-v2-doc-d2q-t5.bm25-default.topics.dl21.txt

tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m map src/main/resources/topics-and-qrels/qrels.dl21-doc.txt runs/run.msmarco-v2-doc-d2q-t5.bm25-default+rm3.topics.dl21.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 src/main/resources/topics-and-qrels/qrels.dl21-doc.txt runs/run.msmarco-v2-doc-d2q-t5.bm25-default+rm3.topics.dl21.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.1000 src/main/resources/topics-and-qrels/qrels.dl21-doc.txt runs/run.msmarco-v2-doc-d2q-t5.bm25-default+rm3.topics.dl21.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m recip_rank -c -m ndcg_cut.10 src/main/resources/topics-and-qrels/qrels.dl21-doc.txt runs/run.msmarco-v2-doc-d2q-t5.bm25-default+rm3.topics.dl21.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

MAP@100                                 | BM25 (default)| +RM3      |
:---------------------------------------|-----------|-----------|
[DL21 (Doc)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)| 0.2387    | 0.2608    |


MRR@100                                 | BM25 (default)| +RM3      |
:---------------------------------------|-----------|-----------|
[DL21 (Doc)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)| 0.8866    | 0.8342    |


nDCG@10                                 | BM25 (default)| +RM3      |
:---------------------------------------|-----------|-----------|
[DL21 (Doc)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)| 0.5792    | 0.5392    |


R@100                                   | BM25 (default)| +RM3      |
:---------------------------------------|-----------|-----------|
[DL21 (Doc)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)| 0.3443    | 0.3580    |


R@1000                                  | BM25 (default)| +RM3      |
:---------------------------------------|-----------|-----------|
[DL21 (Doc)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)| 0.7066    | 0.7572    |

Some of these regressions correspond to official TREC 2021 Deep Learning Track "baseline" submissions:

+ `d_bm25` = BM25 (default), `k1=0.9`, `b=0.4`
+ `d_bm25rm3` = BM25 (default) + RM3, `k1=0.9`, `b=0.4`