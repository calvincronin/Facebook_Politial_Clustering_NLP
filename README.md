# Preface
This is a reduced form of the full paper which can be found in this repo. 

# Abstract
Meta’s Ad Library offers an unprecedented opportunity to more closely examine political marketing trends on one of the largest digital advertising platforms. With this data unsupervised learning applications of Natural Language Processing (NLP) can be leveraged to group advertisement records by political parties. An optimal approach to this task can be deduced through a comprehensive comparison of text vectorization and unsupervised learning models. A reliable model for tying political party affiliation to record-level advertisements would transitively affiliate specific spending and targeting patterns with the same political parties. The aggregate results of which could reveal novel political narratives that exist on a massive scale. Ultimately, of the tested methods, TFIDF text vectorization with K-means clustering resulted in the best model for grouping documents of a given political identity.

# Introduction
The Cambridge Analytica scandal of the 2010’s remains one of the more infamous instances of large scale corporate data abuse. The profiles of tens of millions of Facebook users were harvested without their consent and used to build classification models for microtargeting. The purpose of these models was to more accurately target susceptible users with political advertisements, most notably for the 2016 presidential campaigns of Ted Cruz and Donald Trump.

In the aftermath, Facebook deployed measures for regaining collective user and consumer trust. In an effort to prevent a similar situation and demonstrate transparency, they published their Ad Library. This library makes all current and prior advertisements on Meta owned platforms (Facebook, Instagram, WhatsApp) accessible to the public. Queries return not just the contents of the ad, but the advertising parameters as well. Dates of ad runtime, spending, geographic/demographic targeting, and estimated views are a few examples of fields that could give meaningful insights at the aggregate level.

All that remains is an algorithmic approach to grouping these advertisements into political party affiliations. In this case, the Democrat and Republican parties that comprise the U.S. two-party system. Research in this space is fairly sparse at the moment, and there are no labeled datasets currently available. So the scope of this paper excludes classification models or analyzing advertisements post-grouping. The contribution is instead optimizing combinations of word vectorization and unsupervised learning techniques for the purpose of grouping into expected political identities.

# Data Processing
The data is fairly accessible and allows for query filters to return a specific collection of advertisements. Section 3.1 elaborates on this selection process. While the data is accessible, its contents are unruly. Meta allows for fairly robust customization of its advertisements, so each is a varied mixture of text, images, videos, hyperlinks, etc. Section 3.2 details the processing that is necessary prior to vectorization.

## Data Sourcing
In (Vrancken 2022, 2-3) the structure of the Ad Library and its data are laid out in great detail. There are two primary means of data retrieval. An API and a search bar tool with filtering and sorting options. The objective is to compare methods, not to create a production level tool.Sso the manual search is sufficient here.
### Filters used on search bar queries:
    ● Categorized as “Issues, Elections, or Politics”.
    ● United States based.
    ● Text in English.
    ● Active between 3/1/23 and 4/30/22.
These filters were used to get a small but recent sample of United States political advertising. The “Issues, Elections, or Politics” category returns a number of ads unrelated to the two-party system. So the corpus was eventually composed of separate searches for “Republican” and “Democrat”, both issued with the filters in 3.1.1.

# Methods
The general procedure followed a comprehensive and structured path of comparison. This featured one primary comparison workflow, and two auxiliary efforts. In the primary comparison, the corpus was vectorized with several different methods, each set of vectors underwent dimensionality reduction, and finally each vector was fit to a handful of clustering algorithms. The two auxiliary workflows were Topic Modeling and Hierarchical Clustering, respectively. These were treated as separate workflows because getting comparable results for these requires different processes.

### Primary Comparison Procedures
During this stage, there were four text vectorization methods implemented and the output of each was subjected to two different clustering algorithms for a total of eight sets of results. In between the vectorization and clustering steps, Principal Component Analysis (PCA) was performed on each of the four sets of vectorized text and plotted with data points corresponding to their known political labeling. The plots provide a visual measure of the degree to which the data can be reasonably clustered into expected groups. So the plots aided in tuning vectorization function parameters, as well as contextualizing final results.

### Vectorization Methods
    ● TFIDF
    ● Word2vec
    ● Doc2vec
    ● BERT 5
    
For the implementation of Word2Vec, a custom function was used to generate vectors for each document using a word embedding. For the BERT-based method, a pre-trained sentence transformer model was used.

### Primary Clustering Methods
    ● K-Means
    ● DBSCAN
    
Generally DBSCAN is a more sensitive method. There is a process to derive its ideal parameters, of which small deviations from can result in the model failing to create clusters. It can also be uprooted by input data with a large feature count. So to get the model to work consistently, the principal component vectors were input instead of the actual vectorized text.

For both the text vectorization and clustering methods, there was significant parameter tuning in an effort to maximize clustering of documents according to expected labeling. Each final method was tested with a number of random seeds to ensure consistency in the results.

 ## Topic Modeling
We implemented two topic modeling algorithms, LDA and Top2vec. For LDA, the model was fed the corpus as a BoW. For Top2vec, the topics were generated with a pre-trained transformer word embedding model. In both cases, after the topics were derived, each document was labeled with the topic that it had the highest propensity towards. Then using the proportions of political identity for documents highly related to a given topic, all highly related documents for that topic would be viewed as “Democrat” or “Republican” according to the model.

## Hierarchical Clustering
Hierarchical Clustering was also implemented for all four sets of vectorized text, but without any sort of dimensionality reduction to guide. A cosine similarity was computed for each set of vectors, which was used to define a linkage matrix with ward agglomerative clustering. This matrix was used to create a dendrogram, where documents under a handful of high level hierarchical groupings would be counted as “Democrat” or “Republican” depending on the proportion of actual political identities in that group.

# Results
The general idea behind evaluation was to have a given model infer political identity labels for documents that are grouped together. If there was an obvious majority of a grouping’s actual political identities, the majority identity was assumed for the entire grouping. Evaluation metrics were derived from the total number or correctly inferred document political identities. For example, if a cluster had 12 documents with actual labels of Democrat and 2 documents with actual labels of Republican, the cluster would be assumed to be a democratic one, with 12 correct out of the 14 documents in that cluster. This would be totaled for all groups in a given model’s output, and an overall percent of correctly attributed documents would be calculated. This evaluation metric was calculated regardless of a model's output structure. For example a single model could produce multiple topics or clusters deemed “Republican”. Section 6 goes into more depth on the results shown in this section.

## Text Vectorization and Clustering
Fig 5.1.1 shows the results for the 8 models resulting from combinations of 4 text vectorization approaches with 2 two clustering algorithms.

![image](https://github.com/user-attachments/assets/e5091b36-0dc6-4b8d-8a7a-5f4bd9eff3c7)

In terms of text vectorization, TFIDF produced the best results, followed by BERT. Word2vec and Doc2vec did not exhibit a significantly better ability to segment by political identity than if political identities were guessed at random. K-means seems to generally perform better than DBSCAN. Overall the best performing model was TFIDF with K-means, demonstrating the ability to correctly group 74% of a corpus by political identity.

## Topic Modeling
LDA with BoW failed to generate sufficient topics. Regardless of the number of output topics generated, they were always functionally indistinguishable from each other. The majority of words in one topic were usually present in other topics, and with very similar coefficients. See Appendix Figure 1. As unique topics were never generated, political identities of documents were not inferred. Top2vec with sentence embeddings exhibited an almost identical inability to create distinguishable topics, and groupings were not generated for evaluation.

## Hierarchical Clustering
The hierarchical clustering models produced output structurally different from clustering in section 5.1. Similar to that section, word2vec and doc2vec dendrograms did not successfully group documents with similar political affiliation. But TFIDF and BERT produced dendrograms where some high level sub-hierarchies grouped similarly affiliated documents very well, and other sub-hierarchies did not at all. So the evaluation of these will be slightly different than in 5.1. It is based on an interpretation of the output where hierarchies of ambiguous political identity are removed and the remaining hierarchies are evaluated as in 5.1. For TFIDF and BERT, Figure 5.3.1 shows how much of the output was usable and how well that remaining output grouped documents as expected.

![image](https://github.com/user-attachments/assets/b41d04fe-6af5-4407-a1a5-46954cf144bc)

# Analysis and Interpretation
The results of the overall comparison seemed somewhat unexpected initially, but looking at them retrospectively and in context can help to reconcile these observations. The driving forces behind what worked and what didn’t work form valuable lessons for this sort of endeavor.

Looking first at what didn’t work, all Topic Mapping exercises unmistakably failed. Regardless of text vectorization, transformers, or contemporary methods, all efforts failed to create reasonable topics, let alone group documents well. There are a handful of concepts that could tie to these failures. Topic Mapping is fundamentally different from clustering because the topic - document relationship is not as rigid. Each document can be associated (to varying degrees) with multiple topics. So in retrospect, it was a fairly roundabout and indirect way to go about inferring binary political identities to documents. So it came to be clear that it was ill-fitted to the task at hand. Additionally the models were more restrictive in what they can accept for text vectorization. With the Gensim python library, LDA could only be generated from BoW, and the Top2Vec library only allows for pre-trained word embeddings, so this proved restrictive. Lastly, the dataset was challenging to begin with. Many of the data points were extremely similarly worded, like candidacy announcements. And the dataset was already fairly focused (US Politics, Democrat and Republican) so this could have left little room for sub topics given the relatively small size of the data. It's reasonable given the challenges of working with real world data.

One of the more unexpected failures was the embedding model approach taken with word2vec and doc2vec. For all unsupervised methods tested, they seemed to be just about marginally better than making inferences at random. This was surprising considering the initial expectation that semantically aware embedding models would fare better in dealing with the bidirectionality of different political narratives referencing both themselves and the opposite party. But the model taking into account semantics may not have been ideal in this case.
Looking at the data, ads from republican and democratic sources are actually much more semantically similar than they are different. The majority of the ads were spreading awareness about a given candidate, which were very similar word frequency and semantics wise, regardless of that candidate’s political party. Many never mention the candidate's political party. There were also instances where the ad was politically charged, but the stance was unclear. For example, if the subject was a piece of conservative legislation passing, was the ad celebrating it as a republican or condemning it as a democrat? See Appendix 2 for an example. For these reasons, the data was actually difficult to even manually label, let alone tune an algorithm to do so. Overall it seems like a word embedding approach assimilated the data somewhat and ultimately made it impossible to form cleanly grouped clusters at all.

When you look at the PCA graphs below for TFIDF vs doc2vec, colored by actual political party, you can visually see that one is much easier to cluster by political party than the other.

![image](https://github.com/user-attachments/assets/0c319cbe-fac0-4afb-a00b-19fe3e119feb)
![image](https://github.com/user-attachments/assets/1733d1a2-3b74-4e05-b2f1-fdaa292e7bcb)

# Conclusion
Ultimately K-means with TFIDF resulted in the best unsupervised model to group political advertisement documents by political party. Based on the results, if one were to use this model with k =2, they could expect to group documents with about 74% accuracy. This is a promising result, but not sufficient for the purpose of using clustering to generate a dataset for classification, nor to report metrics on.

![image](https://github.com/user-attachments/assets/7869e69c-111f-4075-ab28-417728982fce)
![image](https://github.com/user-attachments/assets/f3fd1a0d-7f19-48af-8285-78b426e41f95)

