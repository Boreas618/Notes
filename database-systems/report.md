# Report

In this project, we created a lightweight and powerful Infexionplaying agent.

## Approach

In this section, we introduces the algorithm we adopt. To demonstrate why we chose this solution, we present the design alternatives and details of the algorithm. Finally, we delve into the evaluation function.

The main body of the decision-making algorithm is based on the minimax algorithm with $$\alpha-\beta$$ pruning. The search depth is limited to 2.

### Design Alternatives

Initially, we adopted MCTS in our project. However, after testing against a random agent, we found that the 1000-iteration version is extremely time and resource-consuming. This made it impossible for our agent to beat the random agent within the given time and resource limits. We tried different numbers of iterations, but all of them had unacceptable trade-offs between time/resource limits and accuracy. An iteration count of 100 might satisfy the time/resource limit, but it performs close to a random agent. Given a normal board state, there are approximately 50 legal actions to choose from. Assuming we find the optimal exploration constant, each action can only be explored twice. It is unlikely that this version of MCTS will outperform a random agent.

Next, we attempted to train a model based on Convolutional Neural Networks (CNN) to play the game. We treated the task of making decisions based on the current board state as a classification problem. To generate the dataset, we ran Monte Carlo Tree Search (MCTS) on a given board state. The outcome of the MCTS produced a set of candidate actions with their corresponding win rates, such as `{action1: 12/50, action2: 27/50, action3: 40/50, action4: 1/50, ...}`. Each candidate action was considered as a label for the current board state, and the win rate was taken as the confidence of the label. There can be at most $$(49 + 49 \times 6)$$ labels. However, we are not familiar with PyTorch and CNN, so this approach requires further investigation.

We implemented a minimax agent with a search depth of 3. Unfortunately, while the agent is powerful enough against our final implementation, the cost of searching is unacceptable. Frequently, the agent reaches the time limit and loses to the opponent (even a random agent).

In response to the issue of time limits with the minimax approach, we attempted to limit the search time for each iteration. We found that restricting the search to 2 seconds resulted in a low probability of encountering time limits, although this may still occur. However, this solution came at the cost of a significant decrease in search accuracy.

<img src="https://p.ipic.vip/azjacl.png" alt="Screenshot 2023-05-10 at 5.28.57 PM" style="zoom: 25%;" />

In most cases, only approximately 20% of the minimax tree can be explored, which will miss a great number of optimal actions.The agent with a searching depth of 3 and time limit of 2 play 0-5 against our final implementation.

In addition, an experimental greedy agent is proposed. This agent aims to achieve the greatest power difference against its opponent and serves as a benchmark for our final implementation. The agent is fairly simple and fast.

### Design Details

We have implemented the minimax algorithm using $$\alpha-\beta$$ pruning, as presented in the lecture notes. To keep the computation time reasonable, we set the search depth to 2, as explained in the **Design Alternative** section. If the depth is 0 or the state is a terminal state (win/lose/draw), we calculate the evaluation function. The coefficients for the evaluation function are provided by the `situation_awareness()` function, which adjusts them based on the number of turns.

In earlier versions of our design, we included a game-winning search when we had a significant advantage in the number of powers. This was intended to reduce the computation pressure during the final stages of the game. However, as we reduced the search depth, we found that the game-winning search was no more efficient in terms of resources or time than the normal minimax search, and did not improve accuracy. Therefore, we have decided to cancel the game-winning search process.

### Evaluation Function

We came up a couple of features to evaluate the situation:

* `power_difference`

  The number of our powers compared to the opponent.

* `cell_difference`

  The number of cells occupied by us compared to the opponent.

* `position_aggression` (deprecated)

  The distance between the average coordinates of the player and those of their opponent. The closer our cells are to the opponent's, the more aggressive our agent appears.

* `impulsive_penalty` (deprecated)

  We identify the following action as "impulsive": 

  > Suppose we perform a **SPREAD** action to get a cell with a power of 5. However, just adjacent the cell there is a cell which belongs to the opponent. In the next round, it is highly possible that the cell with the power of 5 will be seized by the opponent, which will cause a great loss to the number of powers we have.

  Therefore, a penalty will be given if the state to be evaluated is a result of one or more impulsive actions.

* `clustering` (deprecated)

  The variance of the positions of the cells occupied by us.

There are three deprecated features above that appear to have a negative impact on our agent's performance. The coefficients for the evaluation function are provided by the `situation_awareness()` function, which adjusts them based on the number of turns. In the final implementation of our project, we fine-tuned the strategy as follows:

```python
if turns < 50: # turns_margin
  return 0.5, 0.5 # coe1, coe2
else:
  return 0.5, 0.5
```

However, the best combination of `(turns_margin, coe1_1, coe1_2, coe2_1, coe2_2)` still needs further investigation. The naive fine-tuning process will be presented in the next session.

## Performance Evaluation

In this section, we introduces the performance evaluation statistics of our agent.

**Experiment 1 The tradeoff between power difference and cell difference**

We begin with the following configuration:

```python
if turns < 50: # turns_margin
  return 0.5, 0.5 # coe1_1, coe1_2
else:
  return 0.5, 0.5 # coe2_1, coe2_2
```

**1.1 Set `(core2_1, coe2_2)` to`(0.6, 0.4)` against initial version**

The agent with the configuration `(0.6, 0.4)` plays 6-14 against the initial agent.

**1.2  Set `(core2_1, coe2_2)` to`(0.4, 0.6)` against initial version**

The agent with the configuration `(0.4, 0.6)` plays 8-12 against the initial agent.

**1.3  Set `(core1_1, coe1_2)` to`(0.6, 0.4)` against initial version**

The agent with the configuration `(0.6, 0.4)` plays 7-13 against the initial agent.

**1.3  Set `(core1_1, coe1_2)` to`(0.4, 0.6)` against initial version**

The agent with the configuration `(0.4, 0.6)` plays 6-14 against the initial agent.

It seems that at `(0.5, 0.5)` is the feasible configuration based on the limited amount of experiments. However, we believe that further investigation should be made in order to achieve the best performance.

**Experiment 2 Performance against the greedy agent**

Our agent played 38-2 against the greedy agent, which aims to achieve the greatest power difference against its opponent. This suggests that our agent is powerful enough to compete against some basic algorithms.

**Experiment 3 Decision-making time**

| $$\# turns$$ | $Game\space time(s)$ | $$Average\space time\space per\space turn(s)$$ |
| ------------ | -------------------- | ---------------------------------------------- |
| 161          | 55                   | 0.34                                           |
| 84           | 22                   | 0.26                                           |
| 137          | 48                   | 0.35                                           |
| 249          | 88                   | 0.35                                           |
| 179          | 55                   | 0.31                                           |

The average computing time for each turn is 0.322s, which is extremely fast.

We have observed that there is still a significant amount of available time within the 180-second time limit. Therefore, we can consider extending the searching depth to 3 in some stages of the game to make better use of the available time. 

By following the same method, we can calculate that the average time per move for an agent with a searching depth of 3 is 4.646s, which is nearly 15 times longer than the 2-depth version. Only a small portion of the searches in a game can be replaced with a 3-depth search. Further investigation is needed to determine whether the update can improve performance.

It turns out that our agent can react swiftly to the board state and can beat random and greedy agents. However, the optimal configuration of the agent still needs further fine-tuning.

## Other Aspects

We use NumPy to improve performance.