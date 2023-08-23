# Lexical Analysis

Two roles:

* tokenlization
* classification

## Lexical Analysis

Lexical analysis, often referred to as "scanning," is responsible for dividing a source code into meaningful elements known as **lexemes**. These lexemes encompass an array of entities such as keywords, variable names, operators, and more. 

Unlike the human capacity for identifying these elements via visual cues and contextual understanding, a lexical analyzer does not enjoy this intuition. Therefore, it approaches the code as a sequence of characters and iteratively steps through this string, marking boundaries between distinct lexemes.

However, this process extends beyond mere boundary identification. The lexical analyzer also bears the crucial responsibility of classifying the separated elements according to their semantic roles or token classes.

### Token Classes

Token classes, also known as syntactic categories, form the building blocks of a programming language. These classes may span identifiers, keywords, syntax elements (like open or close parentheses), numbers, and a variety of other forms. Each token class corresponds to a subset of strings permissible within a program's syntax.

- **Identifiers**: These are sequences of characters, beginning with a letter and followed by zero or more alphanumeric characters. Examples of valid identifiers include 'a1', 'f00', or 'b17'.

- **Numbers**: Often represented as an integer or floating-point type, this class typically accepts a non-empty string of digits. Interestingly, strings like '001' or '00' can be valid integers as per this definition.

- **Keywords**: These constitute a predefined set of reserved words in a language, such as 'else', 'if', 'begin', etc.

- **White Space**: Even seemingly inconsequential elements like white spaces, newlines, and tabs merit their own token class.

The objective of lexical analysis is two-fold: classify substrings of the program according to their role (token class), and convey these tokens to the subsequent phase, the parser.

## Lexeme-to-Token Conversion

The lexical analyzer **interface**s with the parser through an iterative process. At each step, it extracts a string (a sequence of characters or bytes) from the source code, and returns a token to the parser. Each token is a pair comprising the token class and the substring from the input.

For instance, given an input string "f00 = 42", the lexical analyzer produces three tokens: an identifier 'f00', an operator '=', and an integer '42'. At this point, it's essential to note that '42' is still a string, denoting an integer in the programming language.

## Tokenization

Tokenization, a core process within lexical analysis, involves breaking down the input string into a sequence of tokens, each of which is characterized by a token class and a substring of the original input. Upon the completion of this process, we have a sequence of tokens, with each token labeled according to its corresponding token class.

In conclusion, lexical analysis is a vital cog in the machine of compiler construction, bridging the gap between human-readable source code and a format that the compiler can process more effectively. The concepts of token classes and tokenization are foundational to understanding the subsequent stages in the compiler's pipeline.

## Examples

### Lexical Challenges in FORTRAN

In FORTRAN, white space is insignificant; meaning that 'VAR1' and 'VA R1' are treated as identical entities. This rule is implemented such that the removal of all white spaces from the program would not change its interpretation.

This unique attribute of FORTRAN creates interesting problems for lexical analysis. For instance, consider a loop keyword 'do'. In the code snippet "DO 5 I = 1, 25", the absence or presence of a comma or a period after the loop iteration variable (i.e., 'I') could significantly alter the program's meaning. If a comma is present, the statement is interpreted as a loop statement; if a period is present, the code is interpreted as an assignment operation where 'DO5I' is a variable name.

These issues require the implementation of lookahead during lexical analysis. Lookahead is a technique where we peek into the input string to make a decision about the current substring. In this context, it means that the lexical analyzer must peek ahead to decide whether 'DO' is a loop keyword or part of a variable name. Despite its usefulness, lookahead can complicate the lexical analysis process, so modern programming languages aim to limit its use.

### Lexical Challenges in PL/I

The Programming Language One (PL/I), designed by IBM in the 1960s, also offers intriguing examples of lexical complexity. Unlike many programming languages, PL/I does not reserve keywords, meaning that keywords can also be used as variable names. This characteristic can lead to code fragments like "IF ELSE THEN = ELSE; ELSE = THEN;", making the lexical analysis rather challenging.

PL/I also exhibits complexities with its 'declare' keyword, which could either represent an array reference or act as a keyword, depending on the larger context of the expression. To determine which interpretation is correct, the lexical analyzer may require unbounded lookahead, scanning the entire argument list and beyond.

### Lexical Challenges in C++

Even modern languages like C++ pose certain lexical problems. Consider an example where we have a nested template operation. If we end up with two consecutive greater-than signs (i.e., '>>'), it can cause lexical ambiguity. This sequence can be interpreted either as two separate close brackets for templates or as a stream operator in C++. While recent compilers have fixed this issue, the earlier solution was to insert a white space between the two greater-than signs.

# Regular Languages

There are several fromailsms for specifying tokens. Regular languages are the most popular.

## Definition of Regular Languages

**Language**: Let alphabet $\sum$ be a set of characters. A language over $\sum$ is a set of strings of characters drawn from $\sum$.

**Languages are sets of strings.** We need some notation for specifying which sets we want.

Regular languages are often specified using regular expressions. Each regular expression denotes a set. 

### Basic Regular Expressions

1. **Single Character Regular Expressions:** 
   
    A regular express: 'C' = {"C"}
    
    {"C"} is a language
    
2. **Epsilon Regular Expressions:** 
   
    $\epsilon$ = {""}

### Compound Regular Expressions

1. **Union Regular Expressions (A + B):** 
   
    This represents the union of two languages represented by A and B. Essentially, it's a set of strings that belong to either language A or language B.
    
2. **Concatenation Regular Expressions (AB):** 
   
    This denotes a language that comprises all possible strings obtained by concatenating a string from language a with a string from language B.
    
3. **Iteration (A*):** 
   
    This represents the set of all possible strings obtained by concatenating 'A' with itself zero or more times. Note that since the iteration count can be zero, the empty string (epsilon) is always an element of A*.

## Regular Expressions over Alphabet 

The regular expressions over an alphabet $\Sigma$ is defined with respect to this alphabet. For instance, we could define a set of regular expressions that include:

1. Epsilon as a regular expression.
2. A single character 'c' as a regular expression, where 'c' is an element of our alphabet.
3. The union of two regular expressions.
4. The concatenation of two regular expressions.
5. The iteration of a regular expression.

## Examples: Building Regular Languages using Alphabet {0,1}

1. **Language: 1*** 

    This denotes all strings of 1s, including the empty string.

2. **Language: (1 + 0)1**

    This denotes all strings obtained by concatenating a string from the language $(1+0)$ with a string from language $1$, i.e., ${11, 01}$.

3. **Language: 0* + 1***

    This is the union of all strings of 0s and all strings of 1s.

4. **Language: (0 + 1)***

    $(0+1)^* = \bigcup_{i \geq0} (0+1)^i=$"", $0+1$, $(0+1)(0+1)$, ... = $\Sigma *$

    This denotes all possible strings of 0s and 1s (including the empty string), or in other words, all strings that can be formed using the alphabet $\{0,1\}$. This is often represented as $\Sigma*$.

Note that a given set of strings can often be represented using multiple, equivalent regular expressions. For instance, the language $(1 + 0)1$ could also be expressed as $11 + 01$.

To conclude, the regular expressions have $5$ constructs:

* Two base cases
* Three compound expressions

# Formal Languages

Regular language is a from of formal language.

In theoretical computer science and compiler design, formal languages play a critical role. A formal language is a set of strings composed of symbols from a specified alphabet. Each formal language is defined by a formal grammar, which provides a set of rules for constructing valid strings in the language.

## Definition

Let $\Sigma$ be a set of characters (an alphabet). A language over $\Sigma$ is a set of strings of characters drawn from $\Sigma$.

## Importance of Alphabet

Different formal languages have different alphabets, which are integral to the definition and interpretation of the language. Without defining the alphabet, we can't discern the strings we're interested in or what the formal language entails.

## The Meaning Function

Formal languages often have a "meaning function," denoted L, that maps the strings in the language to their meanings.

For instance, in regular expressions, a regular expression maps to a set of strings, i.e., the regular language denoted by the regular expression.
$$
L(e) = M
$$
$e$ is the regular expression and $M$ is a set of strings.

Auctually the representation method $\epsilon =$ {""} is not accurate. $\epsilon$ is an expression and {""} is a set of strings. So we'd btter denote this mapping with the meaning function: $L(\epsilon) =$ {""}.

### Syntax vs. Semantics

$L(\epsilon)$ is syntax and {""} is semantics.

Syntax and semantics are in a many-to-one relation. We should let one syntax corresponds to multiple semantics.

# Lexical Specifications

**Keyword:** "if" or "else" or "then" or ...

*'i''f' + 'e''l''s''e'* or short hand *'if' + 'else'*

**Integer**: a non-empty string of digits

*digit = '0' + '1' + '2' + ... + '9'*

In order to achieve non-empty, the regular expression for integer can be denoted as digit digit$^*$ .

Short hand for $AA^*$: $A^+$

**Identifier**: strings of letters or digits, starting with a letter.

*letter = [a-zA-Z]*

So, the identifier can be denoted with *letter(letter+digit)*$^*$

**Whitespace**: a non-empty sequence of blanks, newlines, and tabs.

*(' ' + '\n' + '\t')*$^+$

### unsigned pascal numbers

![Screenshot 2023-07-03 at 5.57.55 PM](https://p.ipic.vip/apxkt6.png)

A more compact way of expressing opt_fraction is *('.'digits)?*.

### Partition the Input

1. Write a rexp for the lexmes of each token class

   **Number** = digit+

   **Keyword** = 'if' + 'else' + ...

   **Identifier** = letter (letter + digit)*

   **Openpar** = '('

   ...

2. Construct **R**, matching all lexmes for all tokens

   R = Keyword + Identifier + Number + ... = $R_1+R_2+\dots$ 

3. Let the input be $x_1\dots x_n$

   For $1\leq i\leq n$ check $x_1\dots x_i \in L(R)$

4. If success, then we know that

   $x_1,\dots x_i \in L(R_j)$ for some $j$

5. Remove $x_1\dots x_i$ from input and go to (3)

**How much input is used?**

"Maximal Munch". Always select the longest match.

**Which toke is used?**

Say that $x_1,\dots x_i \in L(R_j)$ and $x_1,\dots,x_i \in L(R_k)$ both are true.

To be concrete, `if` is both in keyword and identifier.

Priority ordering: Keywords > Identifiers

**What is no rule matches?**

Error strings = all strings not in the lexical specification. We put the error strings last in priority.

# Finite Automata 

A finite automaton consists of 

* An input alphabet $\Sigma$
* A set of states $S$
* A start state $n$
* A set of accepting states $F \subset S$
* A set of transitions **state** $\rightarrow^{input}$ **state**

Transition **s1** $\rightarrow ^a$ **s2** In state **s1** on input **a** go to state **s2**

If the automaton ends in an accepting state when it gets to the end of the input  $\Rightarrow$ accept

Otherwise $\Rightarrow$ reject

Two possible reasons for rejecting: **terminates in state $S \notin F$** or **the automata gets stuck**.

<img src="https://p.ipic.vip/l45vik.png" alt="Screenshot 2023-07-03 at 10.22.58 PM" style="zoom:50%;" />

The language of a finite automaton is a set of accepted strings.

Another kind of transition: $\epsilon-moves$

It can move to a different state without consuming any input. It is a choice so the automata can deceide whether to move ot not.

**Deterministic Finite Automata (DFA)**

* One transition per input per state

* No $\epsilon$-moves

**Nondeterministic Finite Automata (NFA)**

* Can have multiple transitions for one input in a given state
* Can have $\epsilon$-moves

The decisive difference DFA and NFA is that the latter has $\epsilon$-moves. Because the model of multiple transitions per input per state can always be simulated by a slightly more complicated machine with $\epsilon$-move.

![Screenshot 2023-07-03 at 10.50.54 PM](https://p.ipic.vip/u4rnci.png)

A DFS takes only one path through the state graph. An NFA can choose.

An NFA accepts if some choices lead to an accepting state.

NFAs and DFAs recognize the same set of languages. DFAs are faster to execute because there are no choices to consider.

NFAs are in general exponentially smaller.

 # Implementation

Lexical Specifications $\rightarrow$ Regex $\rightarrow$ NFA $\rightarrow$ DFA $\rightarrow$ Tables

## Regex to NFA

<img src="https://p.ipic.vip/syozis.png" alt="Screenshot 2023-07-11 at 6.25.32 PM" style="zoom: 25%;" />

 ## NFA to DFA

**$\epsilon$-closure**: the set of all states including 'S' that can be reached by traversing only ε-transitions, where ε symbolizes an empty string or no-input move.

![Screenshot 2023-07-11 at 6.57.05 PM](https://p.ipic.vip/lhntch.png)

![Screenshot 2023-07-11 at 6.57.20 PM](https://p.ipic.vip/psp2uo.png)

 ## DFA to Table

A DFA can be implemented by a 2D table T. One dimension is "states" and other direction is "input symbol".

<img src="https://p.ipic.vip/850ztj.png" alt="Screenshot 2023-07-11 at 7.10.22 PM" style="zoom:50%;" />

DFAs are faster but less compact.

NFAs are slower but more concise.