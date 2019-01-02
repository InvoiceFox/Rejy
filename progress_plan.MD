*started writing this on 12/20/2018*

### Rejy version 0.01 Alpha

*** basics ***

we will have a functioning rebol-like block interpreter in JavaScript with infix/postfix words, generic methods (fist arg. dispatch), objects with tags, no validation yet, single tag per object. native and user defined functions, generic methods, operators.

DO:
- [x] add postfix evaluator for pipewords
- [x] add infix evaluator for opwords
- [x] add infix evaluator for opwords with more than 2 arguments
- [x] make further plans about modules, object system, write docs
- [x] add tag-word and set-tag-word to parser and evaluator
- [x] add make-tag function with very simple or just no validation dialect
- [x] determine runtime representation of objects with tags (one for now)
- [x] make lit words work
- [x] make tag-word return itself on evaluation
- [x] add tuple function 
- [x] make function that get's value from object via lit-word
- [x] add object function
- [ ] make native generic function word with new tags 
- [ ] add generic or multi function that creates user level generic method

THINK:
- [x] read and write about object system (protoypic but CLOS like)
- [ ] read more about modules / namespaces, try some options
- [ ] read more about scopes (bindology) do we need definitinal scoping, do we want, what would implementation be like, would in 
  slow down things or speed up?
- [ ] how are user functions scoped, how is it implemented
- [ ] look about closures in rebol ... do we want them? as default?

This should be enough to explore made choices (infix/postfix, opwords, pipewords, prototypical objects) start making first simple dialects (validation dialect), test performance, ...

### Rejy version 0.02 Alpha

*** scope, modules, very first dialect ***

Start building the JVM interpreter. Add first dialects to JS version. Add simple parse function. Make simple modules work. Think about Factor vocabularies.

### Rejy version 0.03 Alpha

*** JVM interpreter, first shell scripting ***

Make basic JVM interpterer. Compare performance with JS one. Test features, solidify runtime representations, prepare for first 
practical work for CLI shell scripts.

### Rejy version 0.04 Alpha

*** preparing for web-server ***

Make JS or JVM (if already there) run OK for longer running processes. Test memory consumption, mem. leaks, performance bottlenecks. Start preparing for first webapp-use

### Rejy version 0.05 Alpha

*** first webserver ***

Make JS or JVM (depending on progress and performance results) version web server capable. Combine it with a simple server that 
returns a http response. It could be nodejs or JVM (WAR) based. Web server code will further show 

### Rejy version 0.06 Alpha

*** return to language desing ***

Return to language design, new things that appeared worthy during developments so far. More Rebol features, like more concrete *parse*, deciding on refinements, ports and other Rebol features. 

### Rejy version 0.07 Apha

*** concrete modules, objects, tags ***

If tags show ok concept, make it solid here. make modules solid, objects too. Look at performance too.

***... by this point I am hopefully using it for various shell and some server parts of non-critical apps ... ***

