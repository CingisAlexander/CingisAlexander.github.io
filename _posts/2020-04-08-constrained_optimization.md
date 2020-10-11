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
Optimization belongs to one of the essential parts of machine learning. For instance, it is used to find optimal parameters of a neural network. 
Now, what if the domain of all possible parameters should be constrained? This is the case for support vector machines or backdoor trigger generation. For this reason, *constrained optimization* is of great relevance.

For the backdoor trigger generation I recommend the paper [Neural Cleanse: Identifying and Mitigating Backdoor Attacks in Neural Networks](https://github.com/bolunwang/backdoor)

#### Acknowledgement
Before I start with the explanation I want to give credit to the course (IN2064) Machine Learning held by Prof. Stephan Guennemann at the Technical University of Munich. At this course, the explanation was presented.

#### What is the problem?
Suppose we would like to solve the following constrained optimization problem:

$$
    \begin{align}
        \min_{x} \quad & \overbrace{-(x_1 + x_2)}^{\text{$f_0(x)$}}\\
        \textrm{s.t.} \quad & \underbrace{(x_1^2 + x_2^2)}_{\text{$f_1(x)$}} -1 \leq 0\\
        % &\xi\geq0    \\
    \end{align}
$$

In the Figure below the circle filled with red represents the constrained domain. For now, ignore the further symbols in the Figure.

<p align="center">
<img src="/img/geo.png" alt="geo" width="500" height="500"/>
</p>

There are certain facts about a possible solution $x^*$ of the above constrained optimization problem:
1. If $x^\*$ lies in the interior, this means $(x^\*_1)^2 +(x^\*_2)^2 < 1$. Then, $\nabla f_0(x^\*)= 0$.
2. If $x^\*$ lies on the boundary, i.e. $(x^\*_1)^2 +(x^\*_2)^2 = 1$. Then, $-\nabla f_0(x^\*)= \alpha \nabla f_1(x^\*)$, where $\alpha > 0$.

The gradient in the first fact represents a direction of a steepest ascent. So, $- \nabla f_0(x^\*)= 0$ means from $x^\*$ there is no steepest descent direction left, thus we are already located in a local minimum.

Now, I would like to explain why the second statement holds, by using the [Taylor decomposition](https://en.wikipedia.org/wiki/Taylor_series); additionally let us assume $x^* = x^2$.

We can decompose the $- \nabla f_0(x^\*)$ into to two orthogonal components where one of the component is $\alpha \nabla f_1(x^\*)$. Let $-\nabla f_0(x^\*)= \alpha \nabla f_1(x^\*) + \beta v$ be such an orthogonal decomposition, i.e. $ v^T \nabla f_1(x^\*) = 0$. The decomposition is depicted in the above Figure at the point $x^2$. Further, as depicted in the Figure, let us assume $v \neq 0$.

Using the Taylor decomposition we can decompose $f_0$ at $x^\*$ in the direction $v$:

$$
    \begin{align}
        f_0(x^* + \epsilon v) & \approx f_0(x^*) + \epsilon v^T \nabla f_0(x^{*}) \\
        & = f_0(x^*) + \epsilon v^T (- \alpha \nabla f_1(x^*) - \beta v) \\
        & = f_0(x^*) - \epsilon \beta  v^T v 
    \end{align}
$$

Since $- \epsilon \beta  v^T v < 0$, it follows that $f_0(x^*) - \epsilon \beta  v^T v  < f_0(x^\*)$. Furthermore we know $f_0(x^\* + \epsilon v) \approx f_0(x^\*) - \epsilon \beta  v^T v$; thus, $f_0(x^\* + \epsilon v) < f_0(x^\*)$. So, our assumption about the local minimum ($x^\* = x^2$) is wrong, if we let $\epsilon$ be very small. But is $(x^\* + \epsilon v)$ feasible? Let us check it out using the Taylor decomposition:

$$
    f_1(x^* + \epsilon v) \approx f_1(x^*) + \epsilon v^T \nabla f_1(x^*) = f_1(x^*)  = 0 
$$

Yes it is!

Alright our intital assumption about $x^\*$ was wrong. So, in which direction starting at the point $x^2$ should we go to find $x^\*$?

We know that vector $v$ points in the direction where we can decrease the value of $f_0(x^2)$, i.e. $f_0(x^2 + \epsilon v) < f_0(x^2)$. So, following the direction of the vector $v$ we land at point the $x^3$ depicted in the Figure. At this point, it holds that $-\nabla f_0(x^\*)= \alpha \nabla f_1(x^\*)$, so vector $v$ is zero and we cannot decrease the value of $\nabla f_0(x^\*)$ anymore. Thus, we found a solution to our original problem!

Of course, this explanation does not substitute the proof of the second fact. But, I believe, that the explanation nicely shows why such a condition in case of optimality must hold.