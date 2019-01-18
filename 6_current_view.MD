# Current view

## Truffle, Graal

Our practical focus is on language for backend/server/networking, so Java makes sense. There is Graal and Truffle framework, 
that can greatly speed up the interpreter on it's own, but it's more complex than making regular interpreter. Especially with 
some Rebol-like specifics. You don't know before evaluation itself how many expressions some word will consume:

    add 100 add 200 inc 300
    
The plan is to make a regular interpreter, that dynamically handles all this. For the sake of speed and safety idea is you can
lock values in Rejy and that's the time when user functions could be turned to function / argumets like nodes (and optimized 
by truffle)

## My current language plan

Let's back down on features of the language and experiments. Make a core vith the most important Rejy specifics viable. Language
has some contradicting goals. Maximum (smalltalkish) live editing, environment AND safer, more predictable, more imutable 
production execution. This could be done by locking values (functions, tags, ...), by active precautions like validators, ... OR 
by switch that would turn all lockable functions and values locked when you change mode to production.

 * First I make a very minimal JVM/Truffle interpreter at all and make it work
 * Do some basic benchmarking to see how it behaves (fibonacci), try to make it reasonably fast
 * Add tag / generic methods
 * think about lock 
 
 ## Three levels of Rejy
 
 ### Level 0
 
 We have a parser, that can generate Block, Word, Setword, Number nodes.
 
 We have a evaluator that understands blocks, words, setwords, numbers. It can evaluate builting function object and a user
 function object. All else are functions (if, while, repeat, for, func, ....). For example if is a builting function that accepts number and two blocks. If it's positive it evaluates first block. If it's zero the second.
 
 func is a builtin function that creates a user function object. It's assigned via setword as any other value and when
 reached by evaluator it's invoked.
 
 We do basic optimisation, internal representation, evaluator, function invocation. Think about scoping ...
 
 ### Level 1
 
 Has a Tag object and generic functions. Has prefix and infix words. Here we have a language with some of it's core features.
 
 ### Level 2
 
 Has all the features of a language.
 
 ## First Truffle Implementation
 
Parser is made with Parboiled. It's a PEG parser with runtime / pure Java implementation, so it could also be used as a parse tool inside of Rejy. That's the reason we didnt use parsing generator.

Parser generates most direct Rj*Nodes. Like RjNumber, RjWord, RjSetword, RjBlock. RjBlock has a list of RjNodes, they can be
passed to RjiRootNode which is then used to create a callingContext and run. 

RjiRootNode (interpreted), RjRootNode (The real truffle one) and RjBlockNode could maybe later be turned into 1 class.

In general few things are unclear between what are nodes in interpreter (or AST), what are runtime values and what are nodes in a truffle optimized AST.

RjNumberNode is probably also what runtime would use / return, with it's specialisations, etc. RjBlockNode also ...

But there will be runtime Objects that can't be represented in code, but are only created by builtin functions. 

Like RjFunctionNode. So RjFunctionNode is also never present directly in AST but only a word refering to RjFunctionNode is.

But on the other hand, maybe when we lock the block the functions can be inserted direcly in executable AST ??? And since RjBlockNode is a Node even though it's just internal runtime object, again referenced by a word in AST, not directly on AST (but maybe it could be 
