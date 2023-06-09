# Graph Search

$b$: the maximum branching factor for the search tree

$d$: the depth of the least cost solution

$m$: the maximum depth of the search tree

## Uniformed Search

The standard protocol for finding a plan to get from the start state to a goal state is to maintain an outer **fringe** of partial plans derived from the search tree. We continually **expand** our fringe by removing a node(which is selected using our given **strategy**) corresponding to a partial plan from the fringe, and replacing it on the fringe with all its children.

![Screenshot 2023-02-28 at 7.22.43 PM.png](https://p.ipic.vip/lp4aie.jpg)

The idea of **frindge:**

<img src="https://p.ipic.vip/ssm7vm.png" alt="Screenshot 2023-06-05 at 5.12.10 AM" style="zoom:50%;" />

### Depth-First Search

DFS require a structure that always gives the most recently added objects highest priority. A last-in, first-out stack does exactly this, and is what is traditionally used to represent the fringe when implementing DFS.

Depth-first search simply finds the “leftmost” solution in the search tree without regard for path costs, and so is not optimal.

| Attribute        | Value                                                        |
| ---------------- | ------------------------------------------------------------ |
| Complete?        | No (there are cycles in the graph or the size of the state space is infinite) |
| Time Complexity  | Worst case: explore the entire search tree. $$O(b^m)$$       |
| Space Complexity | $$O(bm)$$maintains _b_ nodes at each of depth levels on the fringe. |
| Optimal?         | No (return first goal state found, not guaranteed to be that of minimal cost) |

**Typological Sort with DFS**

```pseudocode
function TopologicalSort(Graph G):
    visited = {}
    stack = []
    foreach vertex v in G:
        if v not in visited:
            DFS(G, v, visited, stack)
    while stack is not empty:
        print(stack.pop())

function DFS(Graph G, Vertex v, Set visited, List stack):
    visited.add(v)
    foreach vertex w in G.adjacent(v):
        if w not in visited:
            DFS(G, w, visited, stack)
    stack.append(v)
```

### Breadth-First Search

Always select the **shallowest** frindge mode from the start node(As opposed to the deepest node in DFS)

> If we want to visit shallower nodes before deeper nodes, we must visit nodes in their order of insertion.

A first-in, first-out queue.

The time complexity of DFS is terrible in case the $$m$$ is much larger than $$d$$. But is solutions are dense, may be much faster than BFS.

| Attribute        | Value                                                        |
| ---------------- | ------------------------------------------------------------ |
| Complete?        | Yes                                                          |
| Time Complexity  | $$O(b^d)$$ =$$1+b^2+b^3+...+b^d$$                            |
| Space Complexity | $$O(b^d)$$                                                   |
| Optimal?         | Optimal when cost = 1 per step( more detailed statement below); not optimal in general. |

### Uniform Cost Search

To represent the frindge for UCS, the choice is usually a heap-based priority queue, where the weight for a given enqueued node $$v$$ is the path cost from start node to $$v$$, or the backward cost of $$v$$.

A priority queue created this way rearranges itself as we take out the lowest cost path and add its children to maintain the desired order by path cost.

![Screenshot 2023-06-05 at 5.18.13 AM](https://p.ipic.vip/v07gv7.png)

| Attribute        | Value                                                                                                                                                                                                                                     |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Complete?        | Yes. If a goal state exists, it must have some finite length shortest path.                                                                                                                                                               |
| Time Complexity  | ![Screenshot 2023-04-26 at 2.24.38 PM](https://p.ipic.vip/6tq3tc.png)                                                                                                                                                                     |
| Space Complexity | ![Screenshot 2023-04-26 at 2.25.24 PM](https://p.ipic.vip/xrmfg7.png)                                                                                                                                                                     |
| Optimal?         | Yes if assume all edge costs are nonnegative. The strategy is identical to that of Dijkstra’s algorithm and the chief difference is: **UCS terminates upon finding a solution state instead of finding the shortest path to all states.** |

### Iterative Deepening Search

![Screenshot 2023-03-12 at 4.18.08 PM.png](https://p.ipic.vip/682ndp.jpg)

| Attribute        | Value                                                        |
| ---------------- | ------------------------------------------------------------ |
| Complete?        | Yes.                                                         |
| Time Complexity  | $$(d+1)b^0+db^1+(d-1)b^2+\dots+b^d=O(b^d)$$                  |
| Space Complexity | $$O(bd)$$ ( It uses depth-limited search in every search )   |
| Optimal?         | To ensure the optimality of IDS and BFS, the cost function $$g:paths\rightarrow \R$$ **must be non-decreasing with respect to depth.** Alternatively, a stricter requirement would be that the cost is fixed at 1 per step. |

### Bidirectional Search

Complete Time: $$O(b^{\frac{d}{2}})$$ Space: $$O(b^{\frac{d}{2}})$$ It’s optimal if done with correct strategy.

## Informed Search

### Heuristics

The driving force that allow estimation of distance to goal states. They’re functions that take in a state as input and output a corresponding estimate.

Heuristics are typically solutions to **relaxed problems**.

For Pacman example, a common heuristic that’s used to solve this problem is the **Manhattan distance.** $$Manhattan(x_1, y_1,x_2,y_2)=|x_1-x_2|+|y_1-y_2|$$**.**

*   Greedy Search

    Always the lowest heuristic value

    Differences between Greedy Search and UCS: the latter computes _backward cost_, and the former computes **estimated forward cost**.

    | Attribute        | Value                                                        |
| ---------------- | ------------------------------------------------------------ |
    | Complete?        | No (there are cycles in the graph or the size of the state space is infinite) |
| Time Complexity  | Worst case: explore the entire search tree. $$O(b^m)$$       |
    | Space Complexity | $$O(bm)$$maintains _b_ nodes at each of depth levels on the fringe. |
| Optimal?         | No                                                           |
*   $$A^*$$ Search

    | Attribute        | Value                  |
    | ---------------- | ---------------------- |
    | Complete?        | Yes                    |
    | Time Complexity  | $b^{|h-h^*| \times d}$ |
    | Space Complexity | All nodes in memory    |
    | Optimal?         | Yes                    |
    
    Always select the node with **the lowest estimated total cost(from the start node to the goal node)** for exploration. Also uses a priority queue to represent its fringe.
    
    Combines the total backward cost and the estimated forward cost by adding these two values. Yielding an estimated total cost from start to goal.
    
    Complete and optimal given an appropriate heuristic

### Hill-climbing(or gradient ascent/descent)

Local optimal solution

![Screenshot 2023-03-23 at 4.16.27 PM.png](https://p.ipic.vip/rvgeg1.jpg)

### Admissiblity and Consistency

Question: what consistitutes a “good” heuristic

![Screenshot 2023-03-01 at 8.35.42 PM.png](https://p.ipic.vip/kxamhu.jpg)

The condition required for optimality when using A\* tree search is known as **admissibility**. The admissibility constraint states that the value estimated by an admissible heuristic is neither negative nor an overestimate.

$$\forall n, 0\leq h(n)\leq h^*(n)$$, $$h^{*}(n)$$ is the true optimal forward cost to reach a goal state from a given node $$n$$.

> For a relative weighting factor $\omega$: $f(\sigma)=(2-\omega)\cdot g(\sigma)+\omega \cdot h(\sigma)$
>
> We should not overestimate the cost to reach the goal.
> $$
> f(\omega)=(2-\omega)(g(\sigma)+\frac{\omega}{2-\omega}h(\sigma))
> $$
> Thus, $\frac{\omega}{2-\omega}\leq1$

Admissible heuristics can be derived from the exact solution cost of a **relaxed** version of the problem.

**Theorem:** For a given search problem, if the admissibility constraint is satisfied by a heuristic function $$h$$, using $$A^*$$ tree search with $$h$$ on that search problem will yield an optimal solution.

![Screenshot 2023-03-04 at 11.29.46 PM.png](https://p.ipic.vip/rsltto.jpg)

**Graph Search** maintain a “closed” set of expanded nodes while utilizing your search method of choice.

![Screenshot 2023-03-01 at 9.00.35 PM.png](https://p.ipic.vip/fogs09.jpg)

**Note: store the closed set as a disjoint set not a list.**

An additional caveat of graph search is that it tends to ruin the optimality of $$A^*$$.

<img src="https://p.ipic.vip/nz48u8.jpg" alt="Screenshot 2023-03-04 at 10.46.54 PM.png" style="zoom:50%;" />

Because the $$h$$ of $$B$$ is smaller than that of $$A$$, $$C$$ is first expanded as the child of $$B$$ and then put into closed set. Hence, the optimal path will never be found.

To maintain completeness and optimality under $$A^*$$ graph search, an even stronger property than admissibility, **consistency** is needed.

**Enforce not only that a heuristic underestimates the total** distance to a goal from any given node, but also the cost/weight of each edge in the graph. The cost of an edge as measured by the heuristic function is simply the difference in heuristic values for two connected nodes.
$$
\forall A,C\space h(A)-h(C)\leq cost(A,C)
$$
**Theoream.** For a given search problem, if the consistency constraint is satisfied by a heuristic function $$h$$, using $$A^*$$ graph search with $$h$$ on that search problem will yield an optimal solution.

![Untitled](https://p.ipic.vip/p04exn.jpg)

Consistency implies admissibility.

### Dominance

The standard metric to confirm whether a heuristic is better than another is **dominance**.

$$\forall n:\space h_a(n) \geq h_b(n)$$, $$h_2$$ dominates $$h_1$$ and is better for search.

The **trivial heuristic** is defined as $$h(n) = 0$$, and using it reduces $$A^*$$ search to UCS.

# Implementation

Detail: the goal test **on generation** or **on expansion**?

Applying the goal test on generation does not guarantee optimality in general for informed search methods (and UCS) – **there may be lower-cost paths living in the frontier which we have not explored**. However, if we apply the goal test on expansion, all nodes in the frontier will have equal or larger path cost. For uninformed search algorithms (except UCS) this does not affect optimality but will affect time complexity. For example, BFS time complexity becomes $O(b^d+1) $ for goal test upon expansion.
