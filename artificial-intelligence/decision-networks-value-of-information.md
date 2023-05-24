# Decision Networks

**Chance nodes**: Chance nodes in a decision network behave identically to Bayes’ nets

**Action nodes**: Action nodes are nodes that we have complete control over; they’re nodes representing a choice between any of a number of actions which we have the power to choose from. We’ll represent action nodes with rectangles.

**Utility nodes**: Utility nodes are children of some combination of action and chance nodes. They output a utility based on the values taken on by their parents, and are represented as diamonds in our decision networks.

Our goal is to achieve **Maximum Expected Utility**.

## Action Selection in Decision Networks

* Instantiate all evidence 

* Set action node(s) each possible way
* Calculate posterior for all parents of utility node, given the evidence 
* Calculate expected utility for each action 
* Choose maximizing action

# Value of Information

**The value of perfect information** (VPI) mathematically quantiﬁes the amount an agent’s maximum expected utility is expected to **increase** if it observes some new evidence. We can compare the VPI of learning some new information with the cost associated with observing that information to make decisions about whether or not it’s worthwhile to observe.
$$
MEU(e) = \max_{a} \sum_{s}P(s|e)U(s,a)
$$

$$
MEU(e,e^{'}) =\max_{a}\sum_{s}P(s|e,e^{'})U(s,a)
$$

$$
MEU(e,E^{'}) =\sum_{e^{'}}P(e^{'}|e)MEU(e,e')
$$

$$
VPI(E^{'}|e) = MEU(e, E^{'})-MEU(e)
$$

## VPI Proproties

**Nonnegative**
$$
\forall E^{'}, e : VPI(E^{'}|e) \geq 0
$$

$$
VPI(E_j, E_k|e) \neq  VPI(E_j|e)+VPI(E_K|e)
$$