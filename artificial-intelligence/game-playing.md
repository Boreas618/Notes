# Game Playing

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

![Screenshot 2023-04-17 at 11.07.09 PM.png](https://p.ipic.vip/6r05il.jpg)

**Expansion and simulation:**

From the selected leaf, add one child as a new leaf to the tree. Perform playout from the child node, selecting moves according to a playout policy.

**Backpropagation:**

Use outcome of playout to update all ancestor nodes of the new leaf that was added to the tree. Note that if black wins in the playout, we only update the utility (number of wins) for the black nodes, and vice versa if white wins.

![Untitled](https://p.ipic.vip/elkhct.jpg)

Complexity of playout is **linear** in depth of tree, not exponential, since we just pick a move at each level of the playout rather than searching the whole subtree.

We still need a playout policy to pick which moves to take during a playout. Sometimes this policy can be learned by self-play.
