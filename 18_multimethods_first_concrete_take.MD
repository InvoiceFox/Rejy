I was thinking. We have executable / callable objects in form of functions and builtins. We don't want to create new executable
objects and there is no need to. We just need to have a global namespace, that first looks up by the kind of first argument and
then by the word. So we can just bind an existing function or bulitin to this [kind][name] lookup.

So first we need to make this table. Then we make a function for ex. "multi 'name #kind <function>" that assigns it.

We get two namespaces by this. First should be lexical and is used for variables, local functions, etc... second is this one. 
When we evaluate a word we could first lookup in lexical namespace and then this global. But there could be unintended word 
overshadows that could cause problems and make language more bug inviting. And looking up each word in two namespaces is less optimal also.

The first solution to this I came upon is that lexical words are small caps and multi words are capitalized. It makes sense in every
way, just visually I don't that like it ... or ... from rebol lang am not used to it (AT ALL). But it does make sense in every way 
and is less. But most (builtin) functions and everything are multimethods so all words will then be capitalized :/ ... this is not 
that good :(. 

     10 .Greater? 20 |If word Equals? "Janko" { Print "Čuden!" }
     
     
