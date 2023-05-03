# Probability Recap

$$
P(A|B)=\frac{P(AB)}{P(B)}
$$

$$
P(X_1,X_2,\dots X_n) = \prod_{i=1}^{n} P(X_i|X_1,\dots,X_{i-1})
$$

$$X$$,$$Y$$ independent if and only if:
$$
\forall x,y:P(x,y)=P(x)P(y)
$$
$$X$$ and $$Y$$ are conditionally independent given $$Z$$ if and only if:
$$
\forall x,y,z:P(x,y|z)=P(x|z)P(y|z)
$$

# Probabilistic Inference

Our model is a **joint distribution**, i.e. a table of probabilities which captures the likelihood of each possible **outcome**, also known as an **assignment**.

Given a joint PDF, we can compute any probability distribution $$P(Q_1\dots Q_k|e_1\dots e_k)$$ using a simple and intuitive procedure known as **inference by enumeration.**

Three types of variables:

- Query variables $$Q_i$$
- Evidence variables $$e_i$$
- Hidden variables

**Example:** we want to calculate $$P(W|S=winter)$$

![Untitled](https://p.ipic.vip/u99fjt.png)

# BAYES NETS

Not impractical to store the entire joint distribution in the memory.

Each node in the graph represents a single random variables and each directed edge represents one of the conditional probability distributions we choose to store. An edge from node $$A$$ to node $$B$$ indicates that we store the probability table for $$P(B | A)$$. 

There's a conditional distribution for each node. That is a collection of distributions over $$X$$, one for each combination of parents' values.

Only distributions whose variables are absolutely independent can be represented by a Bayes' net without arcs.

<img src="https://p.ipic.vip/qed6ki.png" alt="Screenshot 2023-04-27 at 10.38.06 AM" style="zoom:50%;" />

**Each node is conditionally independent of all its ancestor nodes in the graph, given all of its parents.**

**Example:** for five random variables

- B: Burglary occurs.
- A: Alarm goes off.
- E: Earthquake occurs.
- J: John calls.
- M: Mary calls.

We can construct the Bayes Net according to the causal relationship:

<img src="https://p.ipic.vip/fzesun.png" alt="Untitled" style="zoom:50%;" />

The tail of a directed edge is the cause and the head is considered as the effect.

In the alarm model above, we would store probability tables $$P(B),P(E),P(A | B,E),P(J | A)$$ and $$P(M | A)$$.

Given all of the CPTs for a graph, we can calculate the probability of a given assignment using the **Bayes reconstitution formula** (cumulatively multiply $$X$$ conditioned on its parents):

$$
P(X_1,X_2,\dots ,X_n) =\prod_{i=1}^{n}P(X_i|parents(X_i))
$$

$$
P(−b,−e,+a,+j,−m) = P(−b)·P(−e)·P(+a | −b,−e)·P(+j | +a)·P(−m | +a)
$$

Not every BN can represent every joint distribution.

**BNs need not actually be causal**: correlated but not causal.

Topology really encodes conditional independence.

## Bayes Nets Independence

Important question about a BN: Are two values independent given certain evidence?

* If yes, can prove using algebra
* If no, can prove with a counter example

Each node is conditionally independent of all its ancestor nodes in the graph, **given all of its parents**. But a node may not be independent of its ancestors without the information about its parents.

**Three Basic Cases:**

1. Low Pressure $$\rightarrow$$ Rain $\rightarrow$ Traffic

   According to $$(3)$$, traffic is not independent of low pressure. However,  given Rain, the influence link is "blocked". 

2. Forums busy $$\leftarrow$$ Project Due $$\rightarrow$$ Lab full (Common cause)

   Given Project Due, the Forums and Lab full are independent.

3. Raining $$\rightarrow$$ Traffic $$\leftarrow$$ Ballgame (Common effect)

   Here, however, Raining and Ballgame are independent. Given Traffic,  Raining and Ballgame are not independent. They are in competition as explanation. You know it isn't raining, so it must be a ball game. 

**The general case:**

To get from point A to point B in a graph:

* In a path: all segements needs to be open.

* In a network: one path open is fine.

D-separation is a criterion for deciding whether a set of variables is independent of another set of variables in a Bayes network( i.e. $$X_i\newcommand{\indep}{\perp \!\!\! \perp} \indep X_j |\{X_{k_1}, X_{k_2},\dots,X_{k_n}\}$$ ).

A path is active if each triple is active( i.e. the three basic cases of independent above ). The algorithm checks all undirected paths between $$X_i$$ and $$X_j$$. If one or more active, then independence is not guaranteed. Otherwise, it is. Once $$X$$ and $$Y$$ "d-separated" by $Z$, it means $$X\indep Y | Z$$.

We can prove the local Markov property through the **Bayes reconstitution formula**.

The Local Markov property (a variable is conditionally independent of all other variables given its neighbors) is a necessary condition for the Bayesian reconstitution formula.

## Bayes Nets (Inference)

Inference is the process of calculating the joint PDF for some set of query variables based on some set of observed variables.

Previously, we perform inference by enumeration to compute probability distribution. We can do this by **eliminate** variables one by one. To eliminate a variable $$X$$, we: 

1. Join(multiply together) all factors involving $$X$$.
2. Sum out $$X$$.



![Untitled](https://p.ipic.vip/m92gha.jpg)

## Bayes Nets (Sampling)

An alternate approach for probabilistic reasoning is to implicitly calculate the probabilities for our query by simply counting samples.

We can write a simple simulator for a Bayes Net Model:

<img src="https://p.ipic.vip/0sz0jo.png" alt="Untitled" style="zoom:50%;" />

