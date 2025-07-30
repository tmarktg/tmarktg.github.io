---
layout: post
title: A Deep Dive into Language Implementation
subtitle: Portfolio Project Analysis and Reflection
cover-img: /assets/img/path5.jpg
thumbnail-img: /assets/img/thumb7.jpg
share-img: /assets/img/path5.jpg
tags: [civit.ai, comfyui, fluxgym, generativeai, stable diffusion]
author: Mark Truong
---

For my portfolio project, I developed a fully functional interpreter for a BASIC-like programming language using Python. This was inspired by the advanced programming languages course I took at San Diego State University (SDSU), where I learned core principles of language design, parsing theory, and runtime system implementation. This project encapsulates the end-to-end process of language interpretation — from raw source code to executing programs — and showcases both theoretical and practical computer science expertise.

## Motivation & Academic Inspiration

In the advanced programming languages class at SDSU, we explored fundamental concepts such as lexical analysis, context-free grammars (BNF/EBNF), recursive descent parsing, symbol tables with scoping, and runtime semantics. Topics like operator precedence, type checking, abstract syntax trees (AST), and language design patterns laid the groundwork.
A memorable example was understanding how languages differ between compiled and interpreted implementations, and how syntax and semantics are formally defined. This course also emphasized formal reasoning about language constructs, like loop invariants and axiomatic semantics. The project was a natural extension to solidify these concepts by implementing them hands-on.

Project Overview & Technical Architecture

**Multi-Phase Compiler Architecture**

The interpreter follows the classical multi-phase compiler pipeline:
* Lexical Analysis: Tokenizes the source string into meaningful tokens such as identifiers, literals, operators, and keywords.
* Syntax Analysis (Parsing): Uses a handwritten recursive descent parser to convert tokens into an Abstract Syntax Tree (AST) representing the program’s structure.
* Interpretation: Traverses the AST using the visitor pattern to evaluate expressions, manage state, and execute control flow constructs.

**Advanced Language Features & Type System**
* Dynamic typing supports integers, floats, strings, lists, functions, and booleans.
* Intelligent type coercion and built-in type-checking functions (e.g., IS_NUM(), IS_STR()) mimic real language behaviors.
* Control flow includes full IF/ELIF/ELSE branching, FOR loops with optional steps, WHILE loops, and flow controls like RETURN, BREAK, and CONTINUE.
* Functions are first-class with lexical scoping and closures, demonstrating proper symbol table design with parent-child relationships.

3. Error Handling & Debugging Tools
* Every token and AST node tracks its source code position for precise error reporting.
* Custom error visualization highlights exact error locations using helpful arrows and pointers.
* Contextual error messages include full call stack traces for runtime errors.
* The parser employs recovery mechanisms to continue processing after errors, improving user feedback.

Design Patterns & Software Engineering Excellence
* The Visitor Pattern cleanly separates AST traversal logic, allowing easy extension for new node types or language features.
* The recursive descent parser reflects a deep understanding of parsing theory, including operator precedence, associativity, and look-ahead tokens to resolve ambiguities.
* Modular code structure ensures single responsibility for classes — lexer, parser, interpreter — facilitating maintainability and testing.
* Comprehensive documentation and formal grammar specifications accompany the codebase, reflecting production-quality standards.

Academic Concepts Applied

This project brought to life many concepts from the SDSU course, including:
* BNF/EBNF Grammar: Defining language syntax rules formally, such as <expr> -> <term> {(+ | -) <term>}.
* Lexical vs Syntax Analysis: Tokenization of lexemes versus parsing of tokens into structured ASTs.
* Type Systems and Semantics: Implementing dynamic typing, type coercion, and semantic checks.
* Control Flow Semantics: Modeling branching, looping, and function call semantics precisely.
* Error Handling and Recovery: Position tracking and graceful parsing recovery to handle unexpected input.
* Parser Design: Recursive descent parsing with operator precedence and associativity.
* Symbol Table & Scoping: Nested symbol tables representing variable/function scopes and closures.
* Visitor Pattern: Cleanly separating AST operations from tree structure.

Real-World Skills Demonstrated

This project is a testament to my ability to tackle complex, layered systems involving:
* Compiler/interpreter construction
* Domain-specific language (DSL) design
* Parser and AST manipulation (useful for linters, formatters, syntax highlighters)
* Formal specification and error-driven development
* Modular, maintainable software architecture

Challenges & Learnings
* Designing the recursive descent parser to correctly implement operator precedence and associativity was challenging but rewarding.
* Implementing dynamic typing and proper type coercion required careful thought about runtime behavior.
* Managing error reporting with source position tracking and meaningful messages improved the usability of the interpreter.
* Ensuring the interpreter was extensible for future language features showcased the importance of design patterns like the visitor.

Future Directions

The interpreter’s modular design allows easy extension with features like:
* Optimization passes or bytecode generation for performance
* Garbage collection for memory management
* Additional data types, class/enum definitions, and richer abstractions
* Integration with tooling such as debuggers or IDE plugins

Conclusion

This BASIC interpreter project combines the rich theoretical foundation from my SDSU advanced programming languages class with practical software engineering. It demonstrates a senior-level understanding of language implementation — from lexing and parsing through interpretation — while maintaining clean architecture, comprehensive error handling, and extensibility. The project is a strong portfolio piece illustrating both my academic background and real-world programming skills.

