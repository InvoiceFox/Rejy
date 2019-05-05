I was thinking. We have executable / callable objects in form of functions and builtins. We don't want to create new executable
objects and there is no need to. We just need to have a global namespace, that first looks up by the kind of first argument and
then by the word. So we can just bind an existing function or bulitin to this [kind][name] lookup.

So first we need to make this table. Then we make a function for ex. "
      
      multi 'name #kind <function>
      
... that assigns it.

We get two namespaces by this. First should be lexical and is used for variables, local functions, etc... second is this one. 
When we evaluate a word we could first lookup in lexical namespace and then this global. But there could be unintended word 
overshadows that could cause problems and make language more bug inviting. And looking up each word in two namespaces is less optimal also.

The first solution to this I came upon is that lexical words are small caps and multi words are capitalized. It makes sense in 
every way, just visually I don't that like it ... or ... from rebol lang am not used to it (AT ALL). But it does make sense in 
every way and is less. But most (builtin) functions and everything are multimethods so all words will then be capitalized :/ ... 
this is not that good :(. 

     10 .Greater? 20 |If word Equals? "Janko" { Print "Čuden!" }
     
     
Another version is to reverse it. Let context sensitive words be small caps and let local words be capitalized. Since we have a 
partially stack like language we could say that local variables are "not that desired", so they could be punished by capitalisation 
while context sensitive functions are wellcome and hence small caps.

     Word: "Jim" 10 .greater? 20 |if Word equals? "Janko" { pring "Čuden!" }

I don't like it, but it seems the most obvious way and no less loud (separate characters) (or error prone - no distinciton) alternative.

So maybe we should follow the current opening, keeping in mind that we are seeking better options (if possible).

## How to implement kind

Let's say we have a new type of word - tagword - #tagword. The multiword namespace is a [int][int] namespace where first int is index of a tagword of first parameter and second is the index of the function word.

Native types statically return their tagword indexes which are predefined in code

add 1 2 

getTag of 1 returns index for #integer

are values tagged or are words tagged. I would say values. 

Let's try to start implementing this for simple cases and we see where we get.

