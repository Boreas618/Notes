# Introduction

**Agent:** an entity that has goals or preferences and tries to perform a series of actions that yeild the best/optimal expected outcome given these goals.

**Percept sequence:** the complete history of content the agent has perceived.

**Environment:** the given instantiation of the agent.

**Environment type:**

* Fully vs Partially observable

- Deterministic vs. Stochastic

  The outcome of an action is completely determined by the state of the environment and the action taken by an agent.

- Episodic vs. Sequential

  Episodic environment can be divided into episodes which have no influence on each other

  Short-term actions do not have long-term consequences

- Static vs. Dynamic

  Environment doesn’t change while the agent is deliberating

- Discrete vs. Continuous

  The distinct states of the environmement is finite/infinite

> **Playing soccer**: Partially observable, stochastic, dynamic, sequential, continous
>
> **Shopping for used books on the internet**: Partially observable, stochastic, dynamic, ~~episodic~~ sequential

**World:** an environmant and the agents that reside within it create a world.

**Simple reflex agent**: one that doesn’t think about the consequences of its actions, but rather selects an action based solely on the current state of the world. **Example:** A basic thermostat used in a home to control the heating or cooling system.

**Model-based reflex agent:** maintains some internal state which depends on the percept history, useful if the current environment cannot be fully described by the current percept. **Example:** An autonomous robot vacuum cleaner recording the environment and navigates through the room.

**Utility-based agents:** compares the desirability of different environment states via a utility function. This allows the comparison of different goal states and action sequences and tradeoffs between different goals. **Example:** ADC.

**Goal-based agents**: makes decisions in order to achieve a set of predefined goals, in addition to maintaining internal state.

**Problem-solving agents:**

![](https://p.ipic.vip/6ss42r.png)

**Single-state problem formulation:** complete observable

A problem is defined by four items:

- initial state
- actions(or successor function $S(x)$)
- goal test
- path cost (additive)

A solution is a sequence of actions leading from initial state to goal state.
