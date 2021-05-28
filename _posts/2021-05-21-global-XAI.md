---
layout: post
title: Global eXplainable AI
mathjax: "true"

#![test](/img/geo.png)
---

<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    extensions: ["tex2jax.js"],
    jax: ["input/TeX", "output/HTML-CSS"],
    tex2jax: {
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      displayMath: [ ['$$','$$'], ["\\[","\\]"] ],
      processEscapes: true
    },
    "HTML-CSS": { availableFonts: ["TeX"] }
  });
</script>
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>


### Introduction
In my previous [blog post on XAI](../2020-06-26-XAI), I shed some light on XAI and focused mainly on XAI methods called *local*. However, there also exist a set of XAI methods that is called *global*.

At the beginning of 2021, I successfully submitted my master's thesis about global XAI. So, I believe that it is the right time to write about this exciting XAI domain that is still less researched.

In this blog post, I would like to remind you what a local XAI is. Secondly, present possible definition and forms of global XAI. Finally, I will present a global method that has become promising compared to other researched methods in my thesis.

### Concept of local explainability and SHAP
Usually, XAI are methods that are applied to a model and a sample. For instance, a model could be a neural network that classifies images, and a sample could be an image of a dog or cat. Such methods are also called *local post-hoc explainability methods*

- *explainability methods* are methods that generate an explanation of the model's prediction for a given sample.
- it is called *post-hoc* because the methods are independent of the model and its training procedure.
- it is called *local* because the resulting explanation we obtain is w.r.t. the sample.

Strictly speaking, a local post-hoc explainability method returns an **explainable form** of the model's prediction. Based on the explainable form, we can explain to our self the model's prediction.

In the remainder of this post, I will omit the term post-hoc because all explainability methods are meant to be post-hoc. 

Consider a model $f$ that classifies bird images. We input the image of the bird in [Figure 1](#4) together with $f$ to a local explainability method that returns the explanation on the right-hand side of the figure. The pixels highlighted in red contributed to the classification, and blue-colored pixels decreased the classification probability of a bird.

Based on the explanation, we can recognize that the model classified the image as a bird because of the presence of the beak mainly, which is an intuitive indication of a bird. Therefore, we can say that the classification decision was based on the right indications or features. It even hints that our model has learned the right indicators for a bird classification.

<p align="center">
	<img src="/img/shap_explanation.JPG" alt="geo" width="600" height="300"/>
</p>
<p align="center">
    <em id="4">Figure 1: Figure taken from https://github.com/slundberg/shap (Lundberg & Lee, 2017)</em>
</p>

Suppose another model that identifies a positive or negative sentiment in a text. In [Figure 2](#5), we can see a possible explanation of the sample: *what a great movie!... if you have no taste.*. The text is more a negative review, i.e., it should be classified as a negative sentiment. However, the model considers the word sequence: *great movie* as a strong indication of positive sentiment. We can conclude that our model seems to recognize positive sentiment based on "positive words" and fails at recognizing implicitly hidden negative sentiment, like in [Figure 2](#5).
<p align="center">
	<img src="/img/shap_explanation_transformes.JPG" alt="geo" width="600" height="100"/>
</p>
<p align="center">
    <em id="5">Figure 2: Figure taken from https://github.com/slundberg/shap (Lundberg & Lee, 2017)</em>
</p>

We now have seen local explanations for computer vision and the natural language processing domain. These explanations can be generated using the [*SHapley Additive exPlanations* (SHAP)](https://github.com/slundberg/shap) introduced by [Lundberg & Lee (2017)](#3).

SHAP returns for a given sample $x = (x_1,~\ldots ~,x_M)$ and model $f$ the following explanation

$$
    \begin{align}
        explain(x) = w_0 + \sum_{i=1}^{M}w_i,
    \end{align}
$$

where $w_0 = E[f]$ and the $w_i$ weight corresponds to the importance of feature $x_i$ and is called *feature attribution*.

### Global XAI
In the above examples, we could see which type of indications or features are in a sample used for classification. We might ask: are the indications or features used in general for all samples, or was this indication used specifically in the given sample? This kind of question tries global XAI to answer.

What is a global explanation at all? One could define it as follows: *"Global Explainability attempts to understand the high-level concepts and reasoning used by a model"* [(Bhatt et al., 2020)](#6). I decided to follow in my master's thesis the above definition because one can estimate the general tendentious behavior of a model. This ability is quite useful to recognize undesirable tendentious behavior.

There exist several global explainability methods. In the following, few methods will be mentioned.

[Yang et al. (2018)](#7) proposed *Global Interpretation via Recursive Partitioning* (GIRP) method. The method transforms a model into a decision tree that should reflex the reasoning made in the model, which is also the desired global explanation. The decision tree returned by the method is also called a *surrogate model* [(Arrieta et al. 2019)](#1).

Suppose a significant amount of training images of dogs containing a watermark. This data artifact could cause that the classifier learns during training to classify dog images based on the watermark. Such relation is a well-known problem called *spurious correlations*. [Anders et al. (2019)](#8) developed an algorithm that is able to check if such correlations are present in a trained model.

[Ribeiro et al. (2016)](#9) proposed to obtain a global explanation by selecting instances that have diverse and representative local explanations. Finally, The set of local explanations represents the global behavior of a given model. Furthermore, to estimate such a set [Ribeiro et al. (2016)](#9) introduced the *Submodular Pick* (SP) algorithm. One could call the output also as *explanation by examples* [(Arrieta et al. 2019)](#1).

In my thesis, I proposed several different methods that can be considered global explainability methods for the natural language processing domain. To understand the first method consider a classification model $f$ with two classes $\\{c_1,~c_2\\}$. The method for a class $c \in \\{c_1,~c_2\\}$ generates an artificial textual sample that the model considers as a representative sample of a class *c*. We could check if the model has learned the right features representing the class *c* based on the sample.

The second method is tailored towards feed-forward neural networks. The method aggregates *local dependency explanations* into a graph, where nodes represent words and edges represent relations between the words. The centers of star-shaped subgraphs tendentiously represent important features for the neural network. So, we could see if the neural network has learned the right features by checking the centers.

The method local dependency explanation estimates which feature relations in a sample are important for a model prediction. I proposed this method in my thesis as well.

Based on my conducted experiments, the first proposed method did not work well. We could not recognize high-level concepts or reasoning used by a given model from a generated representative sample. The second method worked well; one could even generate based on the graph adversarial samples. However, the method can only be applied to feed-forward neural networks, which are not so relevant for real-world applications.

The third method *Global Explanation by Aggregating Feature Attributions* based on [van der Linden et al. (2019)](#10) seems to be useful, especially at identifying undesirable tendentious behavior of a model.

### Global Explanation by Aggregating Feature Attributions
Suppose a classifier $f$ that classifies an input $x$ into a class $c \in C$. Further, suppose an explainability method, for instance, SHAP.

In the following, I will describe the steps to obtain a global explanation by aggregating feature attributions for a specific class $c$.

For each sample from a test data, we compute the local explanation. Finally, we stack the explanations into a matrix $W \in \mathbb{R}^{N \times M}$ called an *attribution matrix* where *N* represents the number of samples in a given test data, and *M* represents the number of distinct features $\\{F_1,~\ldots~,F_M \\}$. If a feature $F_j$ does not appear in a test sample $x_i$, then we set $Wij = 0$.

Next we compute for a feature $F_j$ a vector $p_j$ of *normalized importance per class* where

$$
    \begin{align}
        p_{j_c} = \frac{\sqrt{\sum_{i \in S_c} \mid W_{ij} \mid}}{\sum_{k \in C}\sqrt{\sum_{i\in S_k} \mid W_{ij} \mid}},
    \end{align}
$$

where $S_c$ is the set of samples classified as $c$. To find class-specific features [van der Linden et al. (2019)](#10) used the Shannon entropy applied to a $p_j$

$$
    \begin{align}
        H_{j} = - \sum_{c \in C} p_{j_c} \log(p_{j_c})
    \end{align}
$$

To remind low value of $H_j$ means most of the attributions is concentrated in one class. If $H_j$ is large, we have a distribution that is close to uniform-attribution distribution. Further, set
$H_{min} = min(H_1,~\ldots ~, H_M)$ and $H_{max} = min(H_1,~ \ldots ~, H_M)$.

Now, to obtain an estimate of a feature's global importance, we aggregate the local explanations. For a feature $F_j$ we define *global simple importance* as 

$$
    \begin{align}
        I_j^{simple}(c) = \sqrt{\sum_{i \in S_c} \mid W_{ij} \mid},
    \end{align}
$$

In the NLP domain, features that represent common words, like "a", "you", "at", will likely obtain large importances  $I_j^{simple}(c)$, even in cases where the local attributions are in general low. That is why the following scaling factor is defined

$$
    \begin{align}
        \omega(H_j) = \left( 1- \frac{H_j- H_{min}}{H_{max}-H_{min}}\right).
    \end{align}
$$

Finally, we can compute the *global homogeneity-weighted importance* for a feature $F_j$

$$
    \begin{align}
      I_j^H(c) = \omega(H_j) \cdot I_j^{Simple}(c).
    \end{align}
$$

We are still not done. We select based on $I_j^H(c)$ ten most important words to obtain the explanation. Then, embed each token using the  100-dimensional pre-trained GloVe embeddings. Finally, project the selected embeddings into a two-dimensional space using the well-known t-Distributed Stochastic Neighbor Embedding (t-SNE). The visualized clusters of tokens could share a similarity that could be indicative of a class. The similarity could be understood as a possible high-level concept that $f$ uses.

The procedure to compute the *global homogeneity-weighted importance* was proposed by [van der Linden et al. (2019)](#10), and the final global explanation is also based on [van der Linden et al. (2019)](#10).

#### Conducted experiments
In the following, I would like to show a possible result of the above procedure applied to a DistilBERT model that was fine-tuned on Founta dataset [Founta et al. (2018)](#Founta). The use was to train a model that is able to recognize hateful or abusive tweets.

Founta dataset contains tweets, where each tweet is annotated with one of the four classes: {Spam, Abusive, Normal, Hateful}.

<span style="color:red">Since the dataset contains hateful and abusive tweets, we will see in the results of conducted experiments abusive and offensive tweets. The abusive or offensive content does NOT reflex my opinion!</span>

[Figure 3](#global_explanation) represents a global explanation of the fine-tuned distilBERT. For the hateful class, we can recognize a cluster of two words: *Islam* and *Muslim*. This hints that our classifier, in general, seems to use these two words as an indication of hateful content. However, to use these two word as an indication of hateful tweets is not appropriate because there are many non-hateful tweets containing these two words. The figure also shows clear reasoning for how an abusive tweet is identified, i.e., by identifying offensive words. There does not seem to be a clear cluster of words for the spam and normal class that would hint at how these two classes are identified.
<p align="center">
	<img src="/img/visual_global_explanation.JPG" alt="geo" width="600" height="500"/>
</p>
<p align="center">
    <em id="global_explanation">Figure 3: Visualization of ten most important words for each class.</em>
</p>

In total, we can be confident that the DistilBERT model has learned the right features to identify abusive tweets. In the case of hateful tweets, the global explanation does not show a clear strategy that the model uses. However, it hints at a potential undesired behavior, i.e., the model might classify non-hateful tweets related to Islam as hateful tweets more likely.

Okay, that's it! I hope the topic of global XAI is now a bit more clear. If you have any questions or suggestions, then let me know via E-Mail or LinkedIn.

## References
<a id="1">
Arrieta, A. B., Diaz-Rodriguez, N., Del Ser, J., Bennetot, A., Tabik, S., Barbado, A., Benjamins, R., et al. (2020).
Explainable artificial intelligence (xai): Concepts, taxonomies, opportunities and challenges toward responsible ai.
Information Fusion, 58, 82-115.
</a>

<a id="2">
Mosca, E. (2020), Explainability of Hate Speech Detection Models (Master's thesis, Technical University of Munich, Munich, Germany).
</a><br>

<a id="3">
Lundberg, S. M., & Lee, S.-I. (2017). A unified approach to interpreting model predictions.
In Advances in neural information processing systems (pp. 4765-4774).
</a><br>

<a id="6">
Bhatt, U., Xiang, A., Sharma, S., Weller, A., Taly, A., Jia, Y., Ghosh, J., Puri, R., Moura,
J. M. F., & Eckersley, P. (2020). Explainable machine learning in deployment.
Proceedings of the 2020 Conference on Fairness, Accountability, and Transparency,
648–657. https://doi.org/10.1145/3351095.3375624
</a><br>

<a id="7">
Yang, C., Rangarajan, A., & Ranka, S. (2018). Global model interpretation via recursive
partitioning. 2018 IEEE 20th International Conference on High Performance Computing
and Communications; IEEE 16th International Conference on Smart City; IEEE
4th International Conference on Data Science and Systems (HPCC/SmartCity/DSS),
1563–1570.
</a><br>

<a id="8">
Anders, C. J., Marinc, T., Neumann, D., Samek, W., Müller, K.-R., & Lapuschkin, S.
(2019). Analyzing imagenet with spectral relevance analysis: Towards imagenet
un-hans’ ed. arXiv preprint arXiv:1912.11425
</a><br>

<a id="9">
Ribeiro, M. T., Singh, S., & Guestrin, C. (2016). “Why Should I Trust You?”: Explaining
the predictions of any classifier. arXiv, arXiv–1602.</a><br>

<a id="10">
van der Linden, I., Haned, H., & Kanoulas, E. (2019). Global aggregations of local
explanations for black box models. arXiv preprint arXiv:1907.03039.
</a><br>

<a id="Founta">
Founta, A.-M., Djouvas, C., Chatzakou, D., Leontiadis, I., Blackburn, J., Stringhini, G.,
Vakali, A., Sirivianos, M., & Kourtellis, N. (2018). Large scale crowdsourcing
and characterization of twitter abusive behavior. arXiv preprint arXiv:1802.00393.
</a>