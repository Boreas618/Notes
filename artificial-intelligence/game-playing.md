# Game Playing

## Minimax

Idea: choose move to position with highest **minimax value** = best achievable payoff against best play.

<img src="https://p.ipic.vip/oderug.jpg" alt="Screenshot 2023-03-23 at 4.56.42 PM.png" style="zoom:50%;" />

Calculate the minimax value from bottom to top.

<img src="https://p.ipic.vip/oqiw7j.jpg" alt="Screenshot 2023-03-23 at 5.01.53 PM.png" style="zoom:45%;" />

Traverse the search tree all the way down and propagate the minimum and maximum values from the bottom to the top.

**Properties of minimax**

| Attribute        | Value                              |
| ---------------- | ---------------------------------- |
| Complete?        | Yes if tree is finite              |
| Time Complexity  | $$O(b^m)$$ Indeed a DFS            |
| Space Complexity | $$O(bm)$$                          |
| Optimal?         | Yes if against an optimal opponent |

**Resource Limits**

We cannot search all the way down to the leaf in practice.

Standard approach:

- **cutoff test**: depth limit(perhaps add quiescence search)

- **evaluation function**: estimated desirability of position

**Cutting Off Search**

1. $TERMINAL?$ is replaced by $CUTOFF?$
2. $UTILITY$ is replaced by $EVAL$

>  4-ply $\approx$ human novice
>
> 8-ply $\approx$ typical PC, human master
>
> 12-ply $\approx$ IBM’s Deep Blue, Kasparov

**Evaluation Functions**

For chess, typically linear weighted sum o features

$EVAL(s) = \omega_1f_1(s)+\omega_2f_2(s)+\dots+\omega_nf_n(s)$

Exact values don’t matter. Behavior is preserved under any **monotonic** transformation of $EVAL$.

Only the order matters: payoff in deterministic games act as an *ordinal utility* function

### $\alpha-\beta$ **pruning**

Conceptually, alpha-beta pruning is this: if you’re trying to determine the value of a node *n* by looking at its successors, stop looking as soon as you know that *n*’s value can at best equal the optimal value of *n*’s parent.

<img src="https://p.ipic.vip/oderug.jpg" alt="Screenshot 2023-03-23 at 4.56.42 PM.png" style="zoom:50%;" />

As soon as we visit the child of the middle minimizer with value 2, we no longer need to look at the middle minimizer’s other children. Cause the node final value of the middle minimizer should be $$\leq2$$.

Enable us to abandon part of the search tree.

Pruning **does not** affect final result

Good move ordering improves effectiveness of pruning

With “perfect ordering”, time complexity $=O(b^{\frac{m}{2}})$. In this way, the depth of search can be doubled and the agent can easily reach depth 8 and play good chess.

![Screenshot 2023-06-05 at 8.32.57 PM](https://p.ipic.vip/9vvxja.png)

Initially, $$\alpha$$ is $$-\infin$$, $$\beta$$ is $$+\infin$$.

<img src="https://p.ipic.vip/nsxv6r.png" alt="image-20230509122605376" style="zoom:50%;" />

### Algorithm for Nondeterministic Games 

Expectimax gives perfect play

Just like minimax, except we must also handle chance nodes:

if $state$ is a chance node then return average of expectimax of $SUCCESSORS$$(state)$

A version of $α–β$ pruning is possible but only if the leaf values are bounded..

We can do depth-limit in expectimax search.

**Nondeterministic Games in Practice**

* Max nodes (triangle) as in minimax search

* Chance nodes are like min nodes but the outcome is uncertain

  We calculate their expected utilities

The values for the evaluation function **should be** exact. Behaviour is preserved only by **positive linear** transformation of $EVAL$. Hence $EVAL$ should be proportional to the expected payoff.

![Screenshot 2023-04-11 at 10.17.20 PM.png](https://p.ipic.vip/30t7cs.png)

### Assumption vs Reality

The minimax thinks permissively while the expectimax thinks with balance.

<img src="https://p.ipic.vip/3jhjlj.png" style="zoom:50%;" />

## Monte Carlo Tree Search

In some games it can be hard to find a good evaluation function.

Monte Carlo Tree Search was invented as an alternative approach as it does not depend on a heuristic evalution function.

MCTS abandons the use of complete search using a cut-off depth and an evalution function.

Instead, MCTS estimates the average utilty of a state by playing multiple games from that state all the way to a terminal state of the game using an initial move selection policy.

The value of the state is then estimated from the average of the outcomes of those games ( rather than using a heuristic evalution function )

Each game that is played from a state to a terminal state is called a **playout** or **simulation.**

After many playouts from a state, we can esitmate the average utilty of the state( if the only outcome is a win/loss, the average utility is just the win percentage of playouts from that state )

### Key Steps

We gradually expand the search tree using the following four steps:

- Selection

  Starting from the root of the tree, use a selection strategy to choose successor states until we reach a leaf node of the tree.

- Expansion

  From the selected leaf node, expand the node by adding to the tree a single new child from that leaf node.

- Simulation

  Perform a playout from the newly added node to a terminal state, but don’t add the moves/states explored in the playout to the search tree.

- Backpropagation

  Use the outcome from the playout (win/loss) to update the statistics of each node from the newly added node back up to the route.

![Screenshot 2023-04-17 at 10.09.41 PM.png](https://p.ipic.vip/cv9odh.jpg)

### Example

![Screenshot 2023-04-17 at 10.43.30 PM.png](https://p.ipic.vip/h5jlj2.png)

**Selection:**

The selection requires a selection policy to decide which leaf node to expand. A trade-off between exploiting nodes that are known to have a high win percentage versus exploring nodes that have seldom been tested.

---

A selection policy that we can use from reinforcement learning theory is called upper confidence bound applied to trees, and has the formula $$UCB1(n)$$ for a node $$n$$

$$
UCB1(n)=\frac{U(n)}{N(n)} + C\times \sqrt{\frac{\log N(Parent(n))}{N(n)}}
$$

Where

- $U(n)$  is the total utility from all playouts through node $n$
- $N(n)$ is the total number of playouts through node $n$
- $Parent(n)$ is the parent node of $n$ in the tree
- $C$ is a constant that balances the exploitation term $\frac{U(n)}{N(n)}$ and the exploration term $\sqrt{\frac{\log N(Parent(n))}{N(n)}}$.

When choosing between sibling nodes, select the node with the highest $UCB1$ value.

**Expansion and simulation:**

From the selected leaf, add one child as a new leaf to the tree. Perform playout from the child node, selecting moves according to a playout policy.

**Backpropagation:**

Use outcome of playout to update all ancestor nodes of the new leaf that was added to the tree. Note that if black wins in the playout, we only update the utility (number of wins) for the black nodes, and vice versa if white wins.

![Untitled](https://p.ipic.vip/elkhct.jpg)

Complexity of playout is **linear** in depth of tree, not exponential, since we just pick a move at each level of the playout rather than searching the whole subtree.

We still need a playout policy to pick which moves to take during a playout. Sometimes this policy can be learned by self-play.
