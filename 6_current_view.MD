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
 
 ve have a "DO" evaluator. It understands words, setwords, numbers and strings. It can evaluate builting function object and a user
 function object. All else are functions (if, while, repeat, for, func, ....). For example if is a builting function that accepts
 number and two blocks. If it's positive it evaluates first block. If it's zero the second.
 
 func is a builtin function that creates a user function object. It's assigned via setword as any other value and when reached by
 evaluator it's invoked.
 
 ### Level 1
 
 Has a Tag object and generic functions.
 
 ### Level 2
 
 Has all the features of a language.
 
 
