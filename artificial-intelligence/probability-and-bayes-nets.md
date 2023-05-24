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

Not impractical to store the **entire** joint distribution in the memory.

Bayes' Net is a directed, acyclic graph. Each node in the graph represents a single random variables and each directed edge represents one of the conditional probability distributions we choose to store. An edge from node $$A$$ to node $$B$$ indicates that we store the probability table for $$P(B | A)$$.  A condition probability table is stored underneath the node. The CPT is a collection of distributions over $$X$$, one for each combination of parents' values.

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

## Bayes Nets (Independence)

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

   Here, however, Raining and Ballgame are independent without the information of traffic. Given Traffic,  Raining and Ballgame are not independent. They are in competition as explanation. You know it isn't raining, so it must be a ball game. 

**The general case:**

To get from point A to point B in a graph:

* In a path: all segements needs to be open.

* In a network: one path open is fine.

D-separation is a criterion for deciding whether a set of variables is independent of another set of variables in a Bayes network( i.e. $$X_i\newcommand{\indep}{\perp \!\!\! \perp} \indep X_j |\{X_{k_1}, X_{k_2},\dots,X_{k_n}\}$$ ).

A path is active if each triple is active( i.e. the **influence** can flow, the two edges of the triple is **not independent** ). The algorithm checks all undirected paths between $$X_i$$ and $$X_j$$. If one or more active, then independence is not guaranteed. Otherwise, it is. Once $$X$$ and $$Y$$ "d-separated" by $Z$, it means $$X\indep Y | Z$$.

Given a Bayes net structure, we can run d-separation algorithm to build a complete  list of conditional independencies that are necessarily true of the form:
$$
X_i \indep X_j | \{X_{k_1}, \dots X_{k_n}\}
$$
The list determines the set of probability distributions that can be represented.

We can prove the local Markov property through the **Bayes reconstitution formula**.

The Local Markov property (a variable is conditionally independent of all other variables given its neighbors) is a necessary condition for the Bayesian reconstitution formula.

In the Bayes Net, the less number of arrows you draw, the more assumptions of independence you have to make, the fewer joint probability distributions you can represent, the smaller the size of the node. 

## Bayes Nets (Inference)

Inference is the process of calculating the joint PDF for some set of query variables based on some set of observed variables.

Previously, we perform inference by enumeration to compute probability distribution. We can do this by **eliminate** variables one by one. To eliminate a variable $$X$$, we: 

1. Join(multiply together) all factors involving $$X$$.
2. Sum out $$X$$.

For four variables like below, If we want to calculate $$P(T|+e)$$ by inferencing by enumeration, we should get the joint probability with 16 rows. However, with the Bayes Net, we can eliminate the variables one by one.



![Untitled](https://p.ipic.vip/m92gha.jpg)

## Bayes Nets (Sampling)

Why sample?

* Learning: get smaples from a distribution you don't know
* Inference: getting a sample is faster than computing the right answer

**Sampling in Bayes' Nets**

* Prior Sampling

* Rejection Sampling

* Likehood weighting

* Gibbs Sampling

### Prior Sampling

Basic idea:

* Draw $$N$$ samples from a sampling distribution $$S$$
* Compute an approximate posterior probability $$\widehat{P}$$
* Show this converges to the true probaility $$P$$

<img src="https://p.ipic.vip/e0d689.png" alt="Screenshot 2023-05-14 at 9.26.12 PM" style="zoom:50%;" />

```pseudocode
for i in 1, 2, ..., n
	sample(xi, P(Xi|Parents(Xi)))
return (x1,x2,...,xn)
```

Let $$N_{PS}(x_1,\dots,x_n)$$ be the number of samples generated for event $$x_1,\dots, x_n$$
$$
\lim_{N\rightarrow\infin} \widehat{P}(x_1,\dots,x_n) = \lim_{N\rightarrow \infin} {\frac{N_{PS} (x_1,\dots,x_n)}{N}} = S_{PS}(x_1,\dots,x_n) = P(x_1,\dots,x_n)
$$
The sampling procedure is consistent. 

> **Consistent**
>
> An estimator is said to be consistent if it converges in probability to the true value of the parameter being estimated as the sample size increases.

### Rejection Sampling

Let's say we want $$P(C)$$. There's no point keeping all samples around. We just tally counts of $C$ as we go.

```pseudocode
for i in 1, 2, ..., n
	sample(xi, P(Xi|Parents(Xi)))
	if xi not consistent with evidence
		reject
return (x1,x2,...,xn)
```

![Screenshot 2023-05-24 at 8.17.26 PM](https://p.ipic.vip/t8birf.png)

$P(shape|blue)$

If we find a red/green, we just throw it away. Then why did we generate it?

### Likelihood Weighting

<img src="https://p.ipic.vip/fx13gv.png" alt="Screenshot 2023-05-24 at 8.21.48 PM" style="zoom:50%;" />

We force the samples to be blue. Every sample is consistent with the evidence now. The idea here is to fix evidence variables and sample the rest.

But this is not necessarily consistent with the joint distribution. So we introduce the weight.

```pseudocode
w = 1.0

for i = 1, 2, ..., n
	if Xi is an evidence variable
		Xi = observation xi for Xi
		w = w * P(xi|Parents(Xi))
	else
		sample(xi, P(Xi|Parents(Xi))

return (x1,x2,...,xn), w
```

Sampling distribution if $$\bold{z}$$ sampled and $e$ fixed evidence
$$
S_{WS}(\bold{z},e) = \prod_{i=1}^{l}P(z_i|Parents(Z_i))
$$

$$
\omega(\bold{z},e) = \prod_{i=1}^{m} P(e_i|Parents(E_i))
$$

Together, weight sampling distribution is consistent:
$$
S_{WS}(\bold{z},e)\cdot \omega(\bold{z},e)  = \prod_{i=1}^{l}P(z_i|Parents(Z_i)) \prod_{i=1}^{m} P(e_i|Parents(E_i))
$$
Likelihood wieghting doesn't solve all our problems: **Evidence influences the choice of downstream variables, but not upstream ones**

 <img src="https://p.ipic.vip/fzesun.png" alt="Untitled" style="zoom:50%;" />

The problem is, if $$J$$ and $M$ are the evidence, the weight is extremely low in some cases and you can't affect the choice of $A$, $B$ and $E$. But if $A$ and $B$ are the evidence, your simulation is based on the evidence all the way down to the $J$.

<img src="https://p.ipic.vip/v36e3q.png" alt="Screenshot 2023-05-24 at 9.03.27 PM" style="zoom:50%;" />

A simpler would be the above one. Your simluation is like:

```
F A Weight
-f +a 0.01
-f +a 0.01
-f +a 0.01
-f +a 0.01
-f +a 0.01
......
+f +a 0.99
```

### Gibbs Sampling

**Step 1** Fix evidence <img src="https://p.ipic.vip/ysa6wf.png" alt="Screenshot 2023-05-24 at 9.11.38 PM" style="zoom:50%;" />

**Step 2** Initialize other variables **randomly** <img src="https://p.ipic.vip/i1jnu6.png" alt="Screenshot 2023-05-24 at 9.12.15 PM" style="zoom:50%;" />

**Step 3** Repeat 

![Screenshot 2023-05-24 at 9.13.17 PM](https://p.ipic.vip/7e4m5s.png)

Choose a non-evidence variable $X$

Resample $X$ from $P(X|all\space other\space variables)$

![Screenshot 2023-05-24 at 9.15.22 PM](https://p.ipic.vip/189vh2.png)
