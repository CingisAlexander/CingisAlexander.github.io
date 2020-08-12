---
layout: post
title: Intuitive Explanation of Necessary Conditions in Constrained Optimization
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

#### Why optimization?
Optimization belongs to an essential part of machine learning, for instance optimization is used to find optimal parameters of a neural network. 
Now, what if the domain of all possible parameters should be constrained? This is the case in support vector machines, or in backdoor trigger generation. For this reason, *constrained optimization* is from a great relevance.

For the backdoor trigger generation I recommend the paper [Neural Cleanse: Identifying and Mitigating Backdoor Attacks in Neural Networks](https://github.com/bolunwang/backdoor)

#### Credits
Before I start with the explanation I want to give credits to the course (IN2064) Machine Learning held by Prof. Stephan Guennemann at the Technical University of Munich, since at this course the intuitive explanation was presented. 
#### What is the problem?
Suppose we would like to solve the following constrained optimization problem:

$$
    \begin{align}
        \min_{x} \quad & \overbrace{-(x_1 + x_2)}^{\text{$f_0(x)$}}\\
        \textrm{s.t.} \quad & \underbrace{(x_1^2 + x_2^2)}_{\text{$f_1(x)$}} -1 \leq \\
        % &\xi\geq0    \\
    \end{align}
$$

In the image below the circle filled with red represents the constrained domain. For now ignore the further symbols on the image.

<p align="center">
<img src="/img/geo.png" alt="geo" width="500" height="500"/>
</p>

There are certain facts about a possible solution $x^*$:
1. If $x^\*$ lies in the interior, this means $(x^\*_1)^2 +(x^\*_2)^2 < 1$. Then, $\nabla f_0(x^\*)= 0$.
2. If $x^\*$ lies on the boundary, i.e. $(x^\*_1)^2 +(x^\*_2)^2 = 1$. Then, $-\nabla f_0(x^\*)= \alpha \nabla f_1(x^\*)$, where $\alpha$.

The first statement is sort of clear since the gradient represents multidimensonal differentiations, and it also represents the direction of the steepest ascent. So, $- \nabla f_0(x^\*)= 0$ means from $x^\*$ there is no steepest descent direction left, thus we are already located in a local minimum.

Now, I would like to explain why the second statement holds, by using the [Taylor decomposition](https://en.wikipedia.org/wiki/Taylor_series).
Suppose our possible minimum $x^\*$ lies on the boundary. Further, assume for $x^\*$ holds $-\nabla f_0(x^\*)= \alpha \nabla f_1(x^\*) + \beta v$ such that $v^T f_1(x^\*) = 0$ we can decompose the $\nabla f_1(x^\*)$ into to two orthogonal components. This property is depicted on the image at the point $x^2$.
Using the Taylor decomposition we can decompose $f_0$ at $x^\*$ in the direction $v$:

$$
    f_0(x^* + \epsilon v) \approx f_0(x^*) + \epsilon v^T \nabla f_0(x^{*}) = f_0(x^*) + \epsilon v^T (\alpha \nabla f_1(x^*) + \beta v) = f_0(x^*) + \epsilon \beta  v^T v < f_0(x^*) 
$$

From the above equation, it follows that:  $f_0(x^\* + \epsilon v) < f_0(x^\*)$. So, our assumption about the local minimum is wrong, if we let $\epsilon$ very small. One could question, but is $(x^\* + \epsilon v)$ feasible? Let us check it out:

$$
    f_1(x^* + \epsilon v) \approx f_1(x^*) + \epsilon v^T \nabla f_1(x^*) = f_1(x^*)  = 0 
$$

Yes it is!

Coming back to our original problem and observing the image, especially the point $x^2$. We now know that vector $v$ points in the direction where we can decrease the value of $f_0(x^2)$, i.e. $f_0(x^\* + \epsilon v) < f_0(x^\*)$. So, following the direction of the vector $v$ we land at point $x^3$. At this point, it holds that $-\nabla f_0(x^\*)= \alpha \nabla f_1(x^\*)$, so vector $v$ is zero and we cannot decrease the value of $\nabla f_0(x^\*)$ anymore. Thus, we found a solution to our original problem!

#### Generalization
Suppose we have a more general constrained optimization problem:

$$
    \begin{align}
        \min_{x} \quad & f_0(x)\\
        \textrm{s.t.} \quad & f_i(x) \leq 0 & for ~ i=1, \ldots, m\\
    \end{align}
$$

The necessary condition with multiple constraints, in our case we have $m$ different constraints, for $x^\*$ being a local minimum generalizes to the property:

$$
    -\nabla f_0(x^*)= \sum_{i = 1} ^{m} \alpha_i \nabla f_1(x^*), \alpha_i \geq 0.
$$
