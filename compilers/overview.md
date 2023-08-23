# Introduction to Compilers and Interpreters

 This course will provide an in-depth understanding of programming languages and their implementation. We will focus on two major approaches to implementing programming languages: compilers and interpreters, with the majority of the course dedicated to compilers. 

## Interpreters

An interpreter takes as input your program and the data you want to run the program on and produces the output directly. It does not process the program before it executes it on the input. You write the program, invoke the interpreter on the data, and the program immediately begins running. This means that the interpreter is "online", indicating that the work it does is all part of running your program.

## Compilers

A compiler is structured differently. It takes your program as input and produces an executable, another program that might be written in assembly language, bytecode, or any number of different implementation languages. This executable can be run separately on your data to produce the output. The compiler is "offline", as it preprocesses the program first. The compiler serves as a preprocessing step that generates the executable, which can then be run on various inputs or data sets without needing to recompile or process the program again.

## History of Compilers and Interpreters

The development of compilers and interpreters began in the 1950s, with the IBM 704 machine playing a significant role. Despite the high hardware costs, the software costs were even higher. This discrepancy led to efforts to improve programming productivity.

One of the earliest attempts to increase programming efficiency was "speed coding," developed by John Backus in 1953. Speed coding is an early example of an interpreter. While it made program development faster, the programs it produced were ten to twenty times slower than hand-written programs. Furthermore, the speed code interpreter took up a significant amount of memory, which was a concern given the limited memory available in those days.

Despite these limitations, speed coding inspired John Backus to consider scientific computations, which were the most critical applications at the time. He proposed translating formulas into a form that the machine could execute directly, leading to the birth of the Formula Translation Project or FORTRAN Project.

The FORTRAN Project, which lasted from 1954 to 1957, resulted in the development of the first successful high-level language. By 1958, over 50% of all code was written in FORTRAN, indicating the rapid adoption of this new technology. FORTRAN improved programmer productivity and abstraction level, making it easier to leverage the power of machines.

## Impact of FORTRAN

The success of FORTRAN led to a substantial amount of theoretical work and influenced the development of practical compilers. Programming languages require both a deep understanding of theory and good engineering skills, making this field an exciting area of study in computer science.

FORTRAN's structure, which includes five phases - lexical analysis, parsing, semantic analysis, optimization, and code generation - is still preserved in modern compilers. These five phases will be discussed in more detail in the following lectures.

# Lecture Notes: Structure of a Compiler 

Welcome to the second half of our discussion on the structure of a compiler. Let's briefly explore each of its five major phases: lexical analysis, parsing, semantic analysis, optimization, and code generation. We'll draw parallels between these phases and how humans understand English language.

## Lexical Analysis

The initial step in understanding a program, for both a compiler and a human, involves deciphering the words. Humans instinctively recognize separators (e.g., spaces, punctuation, capital letters) which allow us to split a sequence of letters into comprehendible words. This seemingly effortless process is actually a computation task that becomes noticeable when the separators are placed unusually.

In compiler terminology, this process is called lexical analysis, the goal of which is to divide the program text into its "words" or tokens. Tokens can be keywords (e.g., "if", "then", "else"), variable names (e.g., "X", "Y", "Z"), constants (e.g., numbers), operators (e.g., double equals, assignment operator), punctuation (e.g., semi colons), and separators (e.g., spaces).

## Parsing

Once the words are understood, humans move on to understanding the structure of the sentence, a process called parsing. This process involves grouping words into higher-level constructs, often visualized as a tree diagram. Similarly, parsing in compilers involves creating a parse tree for the given code. For instance, an "if-then-else" statement can be broken down into its constituent parts: a predicate, a "then" statement, and an "else" statement.

## Semantic Analysis

After understanding the sentence structure, the next step involves comprehending the meaning of what has been written. This is the equivalent of semantic analysis in a compiler. However, understanding meaning is challenging and often limited in the case of compilers. The main objective of semantic analysis is to catch inconsistencies in the code.

One of the challenges in semantic analysis is dealing with ambiguities, such as variable bindings. Programming languages have strict rules to prevent these types of ambiguities. The scope of variable usage is defined in a way that prevents confusion due to duplicate variable names.

## Type Checking

Compilers also perform type checking as a part of semantic analysis. For instance, if we have a sentence, "Jack looked her homework at home," we can infer a type mismatch between "Jack" and "her," assuming that Jack is male. This is analogous to type checking in programming.

## Optimization

Optimization, the fourth phase in a compiler, doesn't have a direct counterpart in everyday English usage. However, it can be likened to editing. The goal of optimization is to modify the program so that it uses less of a certain resource, such as time, space, power, etc.

In optimization, compilers often apply certain rules to simplify computations, such as replacing multiplication with zero with an assignment of zero. However, this rule may not always be valid, especially in the case of floating-point numbers due to the existence of "not a number" (NaN) in the IEEE floating-point standard.

## Code Generation

The final phase is code generation, often referred to as Code Gen, where the compiler translates high-level programs into assembly code. This is similar to human translation, such as translating English into French.

## Changes Over Time

The proportion of these five phases in a compiler has evolved over time. Early compilers, like FORTRAN I, had fairly complex lexical analysis and parsing phases, a very small semantic analysis phase, and fairly involved optimization and code generation phases.

Modern compilers, however, have much smaller lexical analysis and parsing phases due to the availability of advanced tools. The semantic analysis phase has grown, and the optimization phase is often the dominant component in modern compilers

# Overview of Compiler Structure

We'll recall from our previous discussion that a compiler comprises five major phases: **lexical analysis, parsing, semantic analysis, optimization, and code generation**. Let's delve into each one, explaining them with an analogy to how humans understand English.

## Lexical Analysis

When we read a sentence, the first step is to recognize the words. This process is so automatic that we don't even think about it. However, there's a significant amount of computation going on here. We need to identify separators like blank spaces, punctuation, and clues like capital letters, which help us divide this group of letters into a collection of words that we can understand.

Similarly, the first step for a compiler to understand a program is lexical analysis. The goal of lexical analysis is to divide the program text into its words, or what we call "**tokens**" in compiler terms. Tokens could be keywords, variable names, constants, operators, separators, punctuation, etc.

The challenge in lexical analysis is recognizing composite tokens like double equals `==` as one token rather than two individual equals signs `=`.

## Parsing

Once humans understand the words in a sentence, the next step is to understand the sentence's structure, which we call parsing. In parsing, we group words together into higher-level constructs. For instance, a sentence typically consists of a subject, a verb, and an object.

The process of parsing a program text is akin to parsing an English sentence. For instance, in an "if-then-else" statement, the root of our parse tree will be "if-then-else", which further branches into a predicate, a then statement, and an else statement. 

## Semantic Analysis

After understanding the sentence structure, the next step is to understand the meaning of what has been written. This process, known as semantic analysis, is challenging. While humans can understand meaning intuitively, compilers can only perform a limited kind of semantic analysis. Primarily, compilers try to catch inconsistencies and report errors. 

In English, consider the sentence: "Jack said Jerry left his assignment at home." Here, the word "his" could refer to either Jack or Jerry. Such ambiguity is a significant issue in semantic analysis. In programming languages, the equivalent of this ambiguity is variable bindings. Programming languages have strict rules to prevent such ambiguities.

Semantic analysis in compilers also includes type checking. For instance, the sentence, "Jack looked her homework at home," has a type mismatch between Jack and her, indicating two different people.

## Optimization

Optimization, the fourth phase of a compiler, is somewhat like editing in English. The goal in program optimization is to modify the program so that it uses less of some resource. It could be time (for faster execution), space (for fitting more data in memory), power, network messages, or database accesses.

However, it's crucial to know when it's legal to perform specific optimizations. Some optimizations are valid for certain data types but not for others. 

## Code Generation

Finally, the last compiler phase is code generation, often referred to as Code Gen. This phase can produce assembly code or translate into some other language, akin to human translation from one language to another.

In conclusion, almost every compiler consists of these five phases. However, the proportions have changed significantly over the years. Early compilers like FORTRAN I had complex lexical analysis and parsing phases but a weak semantic analysis phase. Modern compilers have very little lexical analysis and parsing, a fairly involved semantic analysis phase, a dominant optimization component, and a small code-generation phase.

# The Economics of Programming Languages

## Introduction

The economics of programming languages is an intricate topic, mainly driven by two opposing principles: the difficulty of changing an existing language and the relatively low cost of initiating a new one.

## Cost of Starting a New Language

Starting a new language is economically efficient due to the low initial cost of training early adopters. These newer languages can adapt quickly and evolve in response to changing technological trends and application requirements.

## Language Adoption: A Balance of Productivity and Training Cost

Programmers are likely to adopt a new language if the benefits—enhanced productivity—outweigh the training costs. This principle plays a vital role in a programmer's decision between using a well-established, slowly evolving language or adopting a new, rapidly changing one.

## Filling a Void: Emergence of New Languages

As the information revolution progresses, new application domains consistently surface, creating voids that older, established languages may struggle to fill. Consequently, new languages often emerge, better suited to these new domains and capable of addressing their specific needs effectively.

## Family Resemblance: Old Influencing New

New languages often bear a strong resemblance to their predecessors. This similarity not only facilitates a lower training cost but also leverages programmers' existing knowledge, enabling faster and more efficient learning of the new language. The classic example here is Java, designed to appear much like C++ to ease the transition for C++ programmers.

## Evaluating a Good Language: No Universal Metric

A universally accepted metric for evaluating a "good" programming language does not exist. Though some might argue that popularity equates to quality, this belief fails to account for many factors beyond technical excellence that contribute to a language's widespread use.

## Conflicting Needs and Training Costs: Driving Language Evolution

The continual emergence of new programming languages is largely driven by two factors: conflicting application domain needs and the high cost of programmer training. It's often easier and more direct to design a new language to cater to emerging applications than trying to steer an existing language community towards accommodating new trends.

In essence, the evolution of programming languages is reflective of both the changing landscape of application domains and the economic realities of language development and training.