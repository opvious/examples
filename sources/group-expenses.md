# Group expenses

This model finds the optimal way to settle debts within a group of friends.
Optimal here is defined as minimizing the total number of payments sent by each
person and the total money exchanged. The relative importance of each factor is
configurable via an input parameter.

Our formulation has two dimensions:

+ The friends in the group: $\S^d_{friends}:F$.
+ The transactions considered: $\S^d_{transactions}:T$.

There are four parameters:

+ The amount paid by each person during a transaction:
  $\S^p_{paid}: c \in \mathbb{R}^{T \times F}$.
+ Who (and by how much) was involved each person in a transaction:
  $\S^p_{share}: s \in \mathbb{R}_+^{T \times F}$.
+ Settlement trade-off factor: $\S^p_{factor}: f \in \mathbb{R}$. A higher value
  will place a higher importance on minimizing transactions sent per person
  (potentially leading to more money transferred overall).
+ The smallest amount we allow per settlement payment:
  $\S^p_{floor}: g \in \mathbb{R}$.

Our model has one main output:

+ The settlement payment from each person to another:
  $\S^v_{owed[sender,recipient]}: \alpha \in \mathbb{R}_+^{F \times F}$.

For modeling purposes, we also introduce the following variables:

+ A variable capturing whether a person is transferring money to another:
  $\S^v_{transferred[sender,recipient]}: \beta \in \lbrace 0,1 \rbrace^{F \times F}$.
+ A variable capturing the total number of transfers sent by each person:
  $\S^v_{payment}: \gamma \in \mathbb{N}^F$
+ A variable used to cap the number of transfers per person:
  $\S^v_{cap}: \delta \in \mathbb{N}$.

We'll also introduce a few convenience aliases:

+ The total involvement per transaction:
  $\S^a: \forall t \in T, w_t \doteq \sum_{f \in F} s_{t,f}$. This is useful so
  we don't have to require shares to sum up to one.
+ The total paid per transaction:
  $\S^a: \forall t \in T, p_t \doteq \sum_{f \in F} c_{t,f}$.
+ The total paid overall: $\S^a: p^{total} \doteq \sum_{t \in T} p_t$.

We're now ready to express our model's constraints:

$$
\S^c_{zeroSum}: \forall f \in F, \sum_{s \in F} \alpha_{s,f} - \sum_{r \in F} \alpha_{f, r} = \sum_{t \in T}  \left( c_{t,f} - p_{t} \frac{s_{t,f}}{w_t} \right)
$$

$$
\S^c_{transferActivation}: \forall s, r \in F, \alpha_{s, r} \leq p^{total} \beta_{s, r}
$$

$$
\S^c_{paymentActivation}: \forall r \in F, \sum_{s \in F} \beta_{r, s} \leq \gamma_r
$$

$$
\S^c_{capActivation}: \forall f \in F, \gamma_f \leq \delta
$$

$$
\S^c_{aboveFloor}: \forall s, r \in F, \alpha_{s, r} + (1 - \beta_{s, r}) p^{total} \geq g
$$

Finally, our objective:

$$
\S^o: \min f p^{total} \delta + \sum_{s, r \in F} \alpha_{s, r}
$$
