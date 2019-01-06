# Random notes after looking at CLOS / Lisp / Factor

*** WORK IN PROGRESS ***

I got to a collision in goals of generic functions and modules (namespaces / packages). To not reinvent the wheel (and probably doing in in a dumb way) I went looking how CLOS/packages and Factor/vocabularies do it.

Unfortunately, so far I haven't gotten the answer. General instructions and documentation doesn't go that deep to describe combination of these two features. 
Few people I asked also didn't quite answer me. Maybe they don't see the conflict there (maybe I don't understand enough to see
that there isn't) or didn't go as far/deep with generic funtions as I want to go.

While reading about that, I got some ideas about the language that I want to write down. They might be stupid on unformed at this point, but I want to write them down anyway.

## Ideas

### The "uber generic" functions

I haven't gotten the answers about factor / lisp in this regard. Here are 2 related Gists:

https://gist.github.com/InvoiceFox/83b5bc7720d212729495507104818cf7

https://gist.github.com/InvoiceFox/ef08f89b2a127716c47b36907aaaa1ab

***TODO: write down what exactly I see as a problems***

Either way I am slowly forming my version of generic functions, that might or might not be like they are in Factor/Lisp. But I want them to work that way. The simple/primitive reason why I want generic functions for me is: "**I want to use short words (function names)**". Like in natural language where context and subject determines what exactly word means.

Generic words, in my case arent about achieving object orientation goals, but about **words in contexts**. In short you want only 
the context as determinator of which generic method will be used. All additional dispatch rules like namespace is a hindrance and takes away the point. The context is namespaced / imported.

Maybe I should rename this to *context avare functions* to not collide with already defined term *generic funtions*.

### The "uber REPL" - interactive development, serializable environment / state - undo

I was reading about lisp and interactive development. I know lisp has good emacs integration, enables interactive development, but I've never seen how you really **develop** interactively. Maybe you can. I know the interactive shell from rebol, python.

What I would want is a repl where you can fully manipulate the runtime packages, are able to serialize the runtime state to textual code / data in respected files.

Important part of this would be that you can always **UNDO** last few invocations of REPL. This could be achieved if a block of code you eval would accept the clone of current environment and change that one. If you do UNDO you discard the last entry of code and also the latest environment.

If all environment would be serializable even better. Then you could store the environment, load it later and continue. Also
processes could be stopped, moved to new server for example and ran further. Or cloned to multiple servers, etc ...

### The scoping / isolation is still an open question

I want that state mutations are visible and limited. That it's obvious where they happen and where they can't happen. Very raw idea about this is to distinguish 3 or more types of functions. 
 
 * **pure** functions: accepts arguments and returns results based on them. Can only call other pure functions. Same arguments must return same result (Can be cached)
 * **iofu** IO functions: can call other IO or pure functions. Deals with iostream (print), files, network
 * **dirty** Environment / state  functions: can call other state and pure functions. Can change and ask for environment outside itself. and do IO
 * Closures?
 
Would we want to separate Environment to Observers and Mutators? Observers don't change state, but return different result with
same arguments based on state. Environment are basically pure functions that accept whole environment.

Manipulator functions are the ones that accept environment and return a new environment.

Because we don't want to load and store whole environment (like different modules, parent modules, etc, native functions ...) as app state loaded modules would have to be locked in some way and referenced/reloaded as such. Only app-level environment would be serialized.

Can we have local / recursive environment and it's still serialized or should this be limited to some clear state dictionary on top-level???

## Observations

Lisp is dynamically and strongly typed language. The same will be Rejy.

### Scopes in Lisp

Lisp has dynamic and lexical scopes. Dynamical scope behaves similar to REBOL-s definitional scope. What I don't like at it, is 
that you can silently use global variables inside function scope (for example). In Rebol you define local words with refinement 
local in function spec. Which I don't like either - as it's leaking by deafult, not isolated by default.

I would like something isolated by default for variables, but then there is the question, how do you call native or your module
level functions from inside function. Functions are referenced by words, just like other values.

The setwords mechanics themselves mean that functions and other values are the same, so how do we allow something and prevent something else.

For starter we could prevent all modifications to outside scope words, unless explicitly stated. This way we at least prevent
accidental changes to global variables and functions.

Hm ... while function and value words are no different, when evaluator evaluates them it knows if it's a function definition or value. Evaluator could raise an error if we try to getrieve global value where we shouldnt (in pure functions for example) and if function being called isn't pure too.

    aaa: 123
    
    add: pure [ aa bb ] [ aaa + bb ]  // on evaluation raises error because aaa is value defined in outer scope, 
                                      // + is func defined in outer scope , but it passes
    
    addio: func [ aa ] [ print aa ]   // works func is like pure + io
    
    adda: dirty [ aa /read aaa ] [ print getmyid + aa + aaa ]   // works
    
 Function creates it's own context. All variables, except in dirty /write /read are variables limited to this context. That is destroyed after that. **optimisation** maybe we could recycle these context not constantly create and destroy them.
 **crazy idea** what if we make stack dialect and function option that is executed by VM like micro machine. We could then write 
 via stack calls and later also automatically translate normal functions to stack calls and have sort of partial bytecode VM option automatically.
 
    addstac: stackfu [ aa bb -- cc ] [ + print ]
 
 **lexical env** in Rejy ... similar to Lisps. Creates it's own environment that has parent environment as prototype.
 
     x: 200
     a: 100
     let [ x: 1 ] [ print a + x ]       // prints 101
     print x                            // prints 200
 
    

