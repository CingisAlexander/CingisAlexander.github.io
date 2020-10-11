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