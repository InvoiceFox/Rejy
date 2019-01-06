# Random ideas after diving in CLOS / Lisp / Factor

*** WORK IN PROGRESS ***

I got to a collision in goals of generic functions and modules (namespaces / packages). To not reinvent the wheel (and probably doing in in a dumb way) I went looking how CLOS/packages and Factor/vocabularies interact.

Unfortunately, so far I haven't gotten the answer. General instructions and documentation doesn't go that deep to describe this. 
Few people I asked also didn't quite answer me. Maybe they don't see the conflict there (maybe I don't understand enough to see
that there isn't) or didn't go as far/deep with generic funtions as I want to go.

While reading about that, I got some ideas about the language that I want to write down. They might be stupid on unformed at this point, but I think they are at least worth exploring further.

## The "uber generic" functions

I haven't gotten the answers about factor / lisp in this regard. Here are 2 related Gists:

https://gist.github.com/InvoiceFox/83b5bc7720d212729495507104818cf7

https://gist.github.com/InvoiceFox/ef08f89b2a127716c47b36907aaaa1ab

***TODO: write down what exactly I see as a problems***

Either way I am slowly forming my version of generic functions, that might or might not be like they are in Factor/Lisp. But I want them to work that way. The simple/primitive reason why I want generic functions for me is: "**I want to use short words (function names)**". Like in natural language where context and subject determines what exactly word means.

Generic words, in my case arent about achieving object orientation goals, but about **words in contexts**. In short you want only 
the context as determinator of which generic method will be used. All additional dispatch rules like namespace is a hindrance and takes away the point. The context is namespaced / imported.

Maybe I should rename this to *context avare functions* to not collide with already defined term *generic funtions*.

## 
