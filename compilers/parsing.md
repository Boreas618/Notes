# Introduction

Input: sequence of tokens from lexer

Ouput: parse tree of the program

> Example:
>
> * Cool: `if x = y then 1 else 2 fi`
> * Parse input: `IF ID = ID THEN INT ELSE INT FI`
> * Parser output: IF-THEN-ELSE tree 

| Phase  | Input                 | Output           |
| ------ | --------------------- | ---------------- |
| Lexer  | String of  characters | String of tokens |
| Parser | String of tokens      | Parse tree       |

# Context Free Grammars

Not all strings of tokens are programs. Parser must distinguish between valid and invalid strings of tokens. We need a language for describing valid strings of tokens and a method for distinguishing valid from invalid string of tokens.

Programming languages have recursive structure.

An **EXPR** can be if EXPR then EXPR else EXPR fi. Context-free grammars are a natural notation for this recursive structure.

A CFG consists of

* A set of terminals $T$
* A set of non-terminals $N$
* A start symbol $S \space (S\isin N)$ 
* A set of productions $X \rightarrow Y_1 \dots Y_N$ $X\isin N$ $Y_i \isin N \cup T\cup {\epsilon}$

Productions can be read as rules.

For
$$
S\rightarrow (S)
$$
It means the left hand side can be replaced by the right hand side.

-----

1. Begin with a string with only the start symbol $S$.

2. Replace any non-terminal $X$ in the string by the right-hand side of some production $X\rightarrow Y_1\dots Y_n$
3. Repeat (2) until there are no non-terminals

For a string of tokens, we denote them with $\alpha_i$. We follows the steps described above:
$$
\alpha_0 \rightarrow \alpha_1 \rightarrow \alpha_2 \rightarrow \dots \rightarrow \alpha_n
$$
Which can be simplified as  $ \alpha_0 \xrightarrow{*} \alpha_n$.

## Definition

Let G be a context-free grammar with start symbol $S$. Then the language $L(G)$ of $G$ is:
$$
\{\alpha_1\dots\alpha_n | \forall _i \alpha_i \isin T\and S\xrightarrow{*}\alpha_1\dots\alpha_n\}
$$

* Once generated, terminals are permanent.
* Terminals ought to be tokens of the language.

 ## Example

Consider $L_1 = (0\cup1)^*$. An appropriate context-free grammar (CFG) representation is $S\rightarrow \epsilon | S0|S1$.

**Explanation**: For any given string in this language, we can append a '0' or '1' to extend the string, hence generating more sequences within the language.

Now, let's consider a language consisting of balanced parentheses. We can formulate the CFG as follows:

$$
S \rightarrow (S) | \epsilon
$$

In this scenario, $N$, representing the set of non-terminal symbols, contains only $S$, i.e., $N = \{ S\}$. The terminal symbols, denoted by $T$, are the opening and closing parentheses, thus $T = \{(,)\}$.

A fragment of COOL:

* EXPR $\rightarrow$ if EXPR then EXPR else EXPR fi

* EXPR $\rightarrow$ while EXPR loop EXPR pool

* EXPR $\rightarrow$ id

The termianls are in non-cpas.

For simple arithmetic expressions:
$$
E \rightarrow E + E\\
| E * E \\
| (E) \\
| id 
$$
$id$ is a single variable name.

# Derivations

A derivation is a sequence of productions.
$$
S \rightarrow \dots \rightarrow \dots \rightarrow \dots
$$
A derivation can be drawn as a tree.

* The start symbol is the tree's root.
* For a production $X\rightarrow Y_1\dots Y_n$ add children $Y_1\dots Y_n$ to node $X$.

## Example

For a grammar $E \rightarrow E +E | E*E|(E)|id$ and a particular string `id * id + id`. We parse the string and produce a derivation for the string and also at the same time build the tree.

![Screenshot 2023-07-17 at 8.31.16 AM](https://p.ipic.vip/rwie5w.png)

A parse tree has terminals at the leaves and non-termials at the interior nodes. An in-order traversal of the leaves is the original input and the parse tree shows the association of operations, the input string does not.

The example is a left-most derivation. At each step, we replace teh left-most non-terminal.

We have a equivalent right-most derivation. The right-most and left most derovations have the same parse tree.

We are not just interested in whether  $s \isin L(G)$. We need a parse tree for $s$.

# Ambiguity

A grammar is ambious if it has more than one parse tree for some string. Euivalently, there is more than one right-most or left-most derivation for some string.

![Screenshot 2023-07-17 at 8.48.45 AM](https://p.ipic.vip/3m5nvc.png)

There are several ways to handle ambiguity. Most direct method is to rewrite grammar unambiguously.

For example, for the grammar $E \rightarrow E +E | E*E|(E)|id$ above, ambiguity exists. We can rewrite it with:
$$
E \rightarrow E^\prime + E |E^\prime \\
E^\prime \rightarrow id \
$$
