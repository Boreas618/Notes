# Reasoning over Time or Space

Often, we want to r**eason about a sequence** of observations(A changing world).

# Markov Models

 Value of $X$ at a given time is called the **state**

**Structure**: chain

<img src="https://p.ipic.vip/evieqj.png" alt="Screenshot 2023-05-29 at 12.56.00 PM" style="zoom:50%;" />

$X_1$ is a state of the world.

Two functions dictate how the chain behaves: $P(X_1)$ and $P(X_t|X_{t-1})$

Parameters are called **transition probabilities** or dynamic, specify how the state evolves over time (also, initial state probabilities)

Stationarity assumption: transition probabilities the same at all times: $P(X_i|X_{i-1})=P(X_j|X_{j-1})$ $i \neq j$.

Each time step only depends on the previous. This is called the (first order) Markov property.

## Example

<img src="https://p.ipic.vip/flqut7.png" alt="Screenshot 2023-05-29 at 1.08.34 PM" style="zoom:50%;" />

State $X=\{rain,sun\}$

Initial distribution 1.0 $sun$

CPT $P(X_t|X_{t-1})$

| $X_{t-1}$ | $X_t$  | $P(X_t|X_{t-1})$ |
| --------- | ------ | ---------------- |
| $sun$     | $sun$  | 0.9              |
| $sun$     | $rain$ | 0.1              |
| $rain$    | $sun$  | 0.3              |
| $rain$    | $rain$ | 0.7              |

Given $P(X_1)$, $P(x_t)=\sum_{x_{t-1}} P(x_{t-1}, x_t) = \sum_{x_{t-1}} P(x_t|x_{t-1})P(x_{t-1})$.

## Stationary Distributions

For most chains: influence of the initial distribution gets less and less over time. The distribution we end up in is independent of the initial distribution.

The distribution we end up with is called the **stationary distribution** $P_{\infin}$ of the chain. It satidfies $P_{\infin}(X)=P_{\infin+1}(X)=\sum_{x}P(X|x)P_{\infin}(x)$.

What's $P(X)$ at time $t=infinity$?
$$
P_{\infin}(X)=\sum_{x} P(X|x)P_{\infin}(x)
$$
Is actually a linear system.
$$
P_{\infin}(sun) = P(sun|sun) P_{\infin}(sun) + P(sun|rain) P_{\infin}(rain)
$$

$$
P_{\infin}(rain) = P(rain|sun) P_{\infin}(sun) + P(rain|rain) P_{\infin}(rain)
$$

It can be solved (even not linearly independenet).

Alternatively, we run simulation for a long (ideally infinite) time for better resource requirement.

# Hidden Markov Models Formulation

Markov chains are not so useful for most agents. We need observations to update our beliefs. If we want to get the next state, we must have an observation of the current state. 

**Hidden Markov Models (HMMs)** Underlying Markov chain over states X. You observe outputs (effects) at each time step.

<img src="https://p.ipic.vip/9wzf66.png" alt="Screenshot 2023-05-30 at 1.03.11 PM" style="zoom:50%;" />

An HMM us defined by:

* Initial distribution $P(X_1)$

* Transitions: $P(X_t|X_{t-1})$

* Emissions: $P(E_t|X_t)$

HMMs have two important independence properties:

* Markov hidden process: future depends on past via the present
* Current observation independent of all given current state

## Example of a LLM

**Naive stage**: Learn the probability of a word given previous words $P(X_t|X_{t-1})$

**Further**: Learn parameters of a HMM. $P(X_1)$ $P(X_t|X_{t-1})$ $P(E_t|X_t)$

Here, $E$ is a word like "cat". $X$ is a vector we don't know the actual meaning.

**Finally**: Learn probability of a word given all previous words $P(X_i|X_1,...,X_{i-1})$

# Inference

We are given evidence at each time and want to know $B_t(X)=P(X_t|e_{1:t})$

The idea is to start with $P(X_1)$ and derive $B_t$ in terms of $B_{t-1}$.

There are two steps: **Passage of Time** and **Observation**

<img src="https://p.ipic.vip/nxycpj.png" alt="Screenshot 2023-06-03 at 5.55.22 PM" style="zoom:50%;" />

## Passage of Time 

The idea is to introduce a new variable.

The base case is derive $P(X_2)$ from $P(X_1)$
$$
P(x_2) = \sum_{x_1} P(x_1,x_2)=\sum_{x1}P(x_1)P(x_2|x_1)
$$
The generate case:

Assume we have current belief and transition probability:
$$
B(X_t) = P(X_{t}|e_{1:t}) 
$$

$$
P(X_{t+1}|x_t)
$$

Then after one time step passes:
$$
P(X_{t+1}|e_{1:t}) = \sum_{x_t} P(X_{t+1},x_t|e_{1:t})
$$

$$
P(X_{t+1}|e_{1:t}) = \sum_{x_t}P(X_{t+1}|x_t, e_{1:t})P(x_t|e_{1:t})
$$

Given the current state, the next state is independent of all other nodes in the Bayes Net. Therefore,
$$
P(X_{t+1}|x_t, e_{1:t}) = P(X_{t+1}|x_t)
$$
More compactly:
$$
B^{'}(X_{t+1}) = \sum_{x_t} P(X_{t+1}|X_{t}) B(x_t)
$$
The basic idea for this formula is that the beliefs are pushed through the transitions.

## Observation

The base case is to derive $P(X_1|e_1)$ based on $P(X_1)$ and $P(E|X_1)$.

Given $P(X_1)$ and $P(E|X_1)$, how can we get $P(X_1|e_1)$?
$$
P(x_1|e_1) = \frac{P(x_1, e_1)}{P(e_1)}
$$
Here, $P(e_1)$ is a constant. Because:
$$
P(X_1) = \begin{bmatrix}P(x_1) \\ P(x_2) \\ \dots\\ P(x_m)\end{bmatrix}
$$

$$
P(E|X_1) = \begin{bmatrix}P(e_1|x_1)&P(e_1|x_2)&P(e_1|x_3)&\dots&P(e_1|x_m) \\ P(e_2|x_1)&P(e_2|x_2)&P(e_2|x_3)&\dots&P(e_2|x_m) \\ \dots& \dots& \dots& \dots\\P(e_n|x_1)&P(e_n|x_2)&P(e_n|x_3)&\dots&P(e_n|x_m) \end{bmatrix}
$$

$$
P(E|X_1)\cdot P(X_1) = m\cdot \begin{bmatrix}P(e_1) \\ P(e_2) \\ \dots\\ P(e_m)\end{bmatrix}
$$

Thus, $P(x_1|e_1)$ is proportional to $P(x_1, e_1)$ in terms of $x_1$
$$
P(x_1|e_1) = P(x_1)P(e_1|x_1)
$$
And some normalization is needed.

The general case is that assuming we have current belief $B^{'}(X_{t+1})=P(X_{t+1}|e_{1:t})$ and evidence model $P(e_{t+1}|X_{t+1})$. Then, after evidence comes in:
$$
P(X_{t+1}|e_{1:t+1})=\frac{P(X_{t+1},e_{t+1}|e_{1:t})}{P(e_{t+1}|e_{1:t})}
$$
Which is propotional to $P(X_{t+1}, e_{t+1}|e_{1:t})$ in terms of $X_{t+1}$.
$$
P(X_{t+1}, e_{t+1}|e_{1:t})=P(e_{t+1}|e_{1:t}, X_{t+1})P(X_{t+1}|e_{1:t})
$$
Given$X_4$, $E_4$ is conditionally independent of $E_{1:4}$.

Or compactly, $B(X_{t+1}) \propto _{X_{t+1}} P(c_{t+1}|X_{t+1}) B^{'} (X_{t+1})$.

As we get observations, beliefs get reweighed, uncertainty "decreases".

<img src="https://p.ipic.vip/qf8cxr.png" alt="Screenshot 2023-06-03 at 6.39.23 PM" style="zoom:50%;" />

To summarize the above two phases:

Every time step, we start with current $P(X|evidence)$

We then update for time:
$$
P(x_{t}|e_{1:t-1})=\sum_{x_{t-1}}P(x_{t-1}|e_{1:t-1})P(x_t|x_{t-1})
$$
We update for evidence:
$$
P(x_t|e_{1:t}) \propto _{X} P(x_t|e_{1:t-1})\cdot P(e_t|x_t)
$$
This is our updated belief $B_t(X)=P(X_t|e_{1:t})$.

# Particle Filtering

Sometimes $|X|$ is too big to use exact inference. $|X|$ may be too big to even store $B(X)$  (like $X$ Is continuous)

The solution is to use approximate inference.

We just track samples of $X$, not all values. The samples are called particles. Time per step is linear in the number of samples.

Our representation of $P(X)$ is now a list of $N$ particles(samples). Generally, $N << |X|$. 

Once we have sampeld an initial list of particles, the simulation takes a similar form to the forward algorithm, with a time elapse update followed by an observation update at each timestep.

Each particle is moved by sampling its next position from the transition model 
$$
x^{'} = sample(P(X^{'}|x))
$$
This like prior sampling - samples' frequencies reflect the transition probabilities. If enough samples, consistent.

During the observation update for partivle filtering, we use the sensor model $P(E|X)$ to weight each particle according to the probability dictated by the observed evidence and the particle's state.

1. Calculate the weights of all particles as described above.
2. Calculate the total weight for each state.
3. If the sum of all weights across all states is 0, reinitialize all particles.
4. Else, normalize the distribution of total weights over states and resample your list of particles from this distribution.

Rather than tracking weighted samples, we resample. We sample $N$ times from our weighted sample distribution. This is equivalent to renormalize the distribution.

The reason why we do a resample is that some particles' weight are fairly low and donot contribute too much to the our reasoning.

![Screenshot 2023-06-03 at 9.43.30 PM](https://p.ipic.vip/e2652r.png)

