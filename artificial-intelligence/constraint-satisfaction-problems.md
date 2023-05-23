# Constraint Satisfaction Problems

CSPs are a type of identification problem, problems in which we must simply identify whether a state is goal state or not, **with no regard to how we arrive at that goal.**

CSPs are deﬁned by three factors:

1. **Variables** - CSPs possess a set of $N$ variables $X_1 ,...,X_N$ that can each take on a single value from some deﬁned set of values.
2. **Domain** - A set $\{ x_1 ,...,x_d \}$ representing all possible values that a CSP variable can take on.
3. **Constraints** - Constraints deﬁne restrictions on the values of variables, potentially with regard to other variables.

NP-hard There exists no known algorithm for finding solutions to them in polynomial time.

Usually represented as constraint graphs.

- Unary constraints
- Binary constraints: each constraint relates at most two variables
- Higher-order constraints
- Preferences (soft constraints)

## Applying Standard Search

**Initial state**: the empty assignment

**Successor function**: assign a value to an unassigned variable that does not conflict with current assignment. 一 fail if no legal assignments (not fixable!) 

**Goal test**: the current assignment is complete
$b=(n-l)\times d$ at depth $l$, hence $n!d^n$ leaves.

# Solving Constraint Satisfaction Problems

Depth-first search for CSPs with single-variable assignments is called **backtracking** search.

Backtracking search: 

- Fix an ordering for variables, and select values for variables in this order.
- When selecting values for a variable, only select values that don’t confict with any previously assigned values.

![Untitled](https://p.ipic.vip/bycxcz.jpg)

Three ideas to improve backtracking:

- Ordering
- Filtering: detect inevitable failure
- Structure

## Filtering

A naive method of filtering is **forward checking**.

Whenever a value is assigned to a variable $X_i$, prunes the domains of unassigned variables that share a constraint with $X_i$ that would violate the constraint if assigned.

The idea of forward checking can be generalized into the principle of **arc consistency**.

An arc $X\rightarrow Y$ is consistent if for every $x$ in the tail there is some $y$ in the head which could be assigned without violating a constraint.

Key point: delete from the tail

Given an assignemnt. Try to make all the arcs consistent.

If $X$ loses a value, neighbours of $X$ should be rechecked! 

Arc consistency check detects failure earlier than forward checking.

Arc consistency check can be run as a preprocessor or after each assignment.

- Begin by storing all arcs in the constraint graph for the CSP in a queue $Q$.
- Iteratively remove arcs from $Q$ and enforce the condition that in each removed arc  $X_i\rightarrow X_j$ , for every remaining value $v$ for the tail variable $X_i$ , there is at least one remaining value $w$ for the head variable $X_j$ such that $X_i = v$, $X_j = w$ does not violate any constraints. If some value $v$ for $X_i$ would not work with any of the remaining values for $X_j$ , we remove $v$ from the set of possible values for $X_i$ .
- If at least one value is removed for $X_i$ when enforcing arc consistency for an arc  $X_i\rightarrow X_j$ , add arcs of the form  $X_k\rightarrow X_i$ to $Q$, for all unassigned variables $X_k$ . If an arc  $X_k\rightarrow X_i$ is already in $Q$ during this step, it doesn’t need to be added again.
- Continue until $Q$ is empty, or the domain of some variable is empty and triggers a backtrack.

Arc consistency is typically implemented with the AC-3 algorithm:

![Untitled](https://p.ipic.vip/hlslll.jpg)

A worst case of time complexity: $O(ed^3)$ where $e$ is the number of arcs and $d$ is the size of the largest domain. 

Fewer backtracks and assignment, more computation.

Arc consistency is a subset of a more generalized notion of consistency known as **k-consistency**. Arc consistecy is 2-consistency.

Our consistency is about **pair** here. 

**2-Consistency:** for each pair of nodes, any consistent assignment to one can be extended to the other.

**K-Consistency:** for any set of k nodes in the CSP, a consistent assignment to any subset of $k − 1$ nodes guarantees that the $k$ th node will have at least one consistent value.

**Strong K-consistency:** A graph that is strong k-consistent possesses the property that any subset of k nodes is not only k-consistent but also $k − 1,k − 2,...,1$ consistent as well.

Strong n-consistency means we can solve without backtracking.

## Ordering

Principle: 

- **Minimum Remaining Values(MRV)**: When selecting which variable to assign next, using an MRV policy chooses whichever unassigned variable has the fewest valid remaining values (the most constrained variable).
- **Degree Heuristic**: choose the variable with the most constraints on remaining variables.
- **Least Constraining Value (LCV)**: When selecting which value to assign next, a good policy to implement is to select the value that prunes the fewest values from the domains of the remaining unassigned values.

In terms of choosing variable, we take the most constrained variable.

In terms of choosing value, we take the least constrained variable.

Combing the principles can make some relative big problems feasible.

## Structure

Extreme case: independent subproblems

For a tree-structured CSP, we can reduce the runtime for finding a solution from $O(d^{N})$ all the way to $O(nd^2)$

<img src="https://p.ipic.vip/wx0m31.jpg" alt="Untitled" style="zoom:50%;" />

Tree-structured CSP algorithm:

- Pick a root node
- Linearize the resulting directed acyclic graph
- Perform a **backwards pass** of arc consistency
- Perform a **forward assignment**

The tree structured algorithm can be extended to CSPs that are reasonably close to being tree-structured with **cutset conditioning**. It involves first finding the samllest subset of variables in a constraint graph such that their removal results in a tree(such a subset is known as a cutset for the graph).

![Untitled](https://p.ipic.vip/6njv7z.jpg)

THe runtime of cutset conditioning on a general CSP is $O(d^c (n-c)d^2)$

## Local Search

Local search works by iterative improvement - starting with some random assignment to values then repeatedly selecting the variable that violates the most constraints and resetting it to the value that violates the fewest constraints (a policy known as the **min-conﬂicts heuristic**)

Local search appears to run in almost constant time and have a high probability of success not only for N-queens with arbitrarily large N, but also for any randomly generated CSP.

But incomplete and suboptimal. Sometimes expensive.
