# Looking at factor, unix shell, lisps ...

> These are great languages hundreds levels above my toy interpreter, and people working on them were ten levels above me. I can 
> still have opinions and write them down, for my sake at least, but have the first scentence 
> in mind when you are reading this.

### Factor - a stack based language

I used to program in factor for about a year 10 years back. The stack approach certanly has it's magic. Most of it can't be seen from a cursory look. A Factor in particular, was adding very advanced and cool solutions and tools to the mix. I still think Slava Pestov (now he works on Swift team) and other people around it were total geniuses.

**PRO:** these languages reject **variable names**. And many times variables are just unnecesary noise that breaks the flow on logic. This is some factor code:

    "Hello, world" print                                            ; prints "Hello, world"
    
    { 4 8 15 } [ 2 * ] map .                                        ; -- puts { 8 16 30 } on stack
    
    "A man, a plan, a canal: Panama." [ Letter? ] filter >lower     ; -- puts amanaplanacanalpanama on stack
    
    "http://factorcode.org" http-get nip string>xml                 ; loads page, turns it into XML nodes
    "a" deep-tags-named                                             ; finds all a tags
    [ "href" attr ] map                                             ; maps list of tags to list of href attrs
    [ print ] each                                                  ; prints each of links

**CON:** the language introduces stack shuffling words, which is even bigger line noise

**PRO:** advanced solutions were invented to mitigate the need for stack shuffling words (combinators, nice tuples,...)

**CON:** with all the advanced solution, language maybe became to wide - to powerful for it's own good

**PRO:** the postfix sytax is naturaly oriented as the operations flows in a program

> Code written in this style, in which a single input value
> is gradually transformed and reshaped in distinct steps into
> a result, is known as pipeline code, named due to the resem-
> blance to the use of pipes in Unix shells. Pipeline code is
> expressed very naturally in Factor; given several words, say
> a, b, c, each taking a single object from the stack and push-
> ing a single result, the code  that applies each word to the
> result of the previous word is simply written as:

    10 a b c            ; -- factor code
    
    c(b(a(10)))         ; -- python code
    
    (c (b (a 10)))      ; -- lisp code
    
    c b a 10            ; -- rebol code
    
    c b a 10            ; -- Rejy code
    10 |a |b |c         ; -- Rejy code using pipewords
    
**PRO:** the object system, mainly the generic methods was great idea, inspired by Dylan and CLOS (more about it lower)

**PRO:** the module system disjoint from type system is another great one. Quote:

> Whereas many languages, notably Java, combine their module system and type system, Factor separates the two concepts
> to  maximize  flexibity  and  modularity  of  code. Vocabularies provide  modularity,  source  code  organization,
> and namespaces. Independent of source code organization, classes and generic words organize the data types and operations
> of a program at run time.

Quotes come from - a great read: http://factorcode.org/littledan/dls.pdf


### Bash - unix shell

Unix shell is an awesome composable environment **that works**. There you can compose installed binaries (awk, sed, grep), local files and your scripts via pipes, xargs, etc ...

**PROs:** it works! it's infinitely versatile and extensible! great "repl" (shell), limitles runtime (whole computer / OS)

    cat invs201810.txt | grep -P $"\tpe\t|\tp\t" | cut -f1 | xargs -i ./downloadone invoice-sent {} pdf
    
    cat dls201810.txt | grep -P $"\tpe\t|\te\t" | head -n100  | awk '{xx= "./send-email invoice-sent " $1 " " $3; system(xx)}' > log.txt

**CONs:** it's incoherent, it works on raw text (no structure)

**CON:** for deeper stack problems you are using named pipes which are somewhat cumbersome

```
# find partner, create invoice for it, add 3 bodies, download pdf and email odt
$ mkfifo _P1 _P2 _P3 _P4
$ invf contact find "Alba co" -justid -1 | \
      xargs -I contact_id \
      invf inv create -c contact_id | \
      tee _P1 | tee _P2 | tee _P3 | tee _P4 | \
      xargs -I inv_id \
      invf inv-b add inv_id -d "Programming" -u hour -q 30 -p 40 -t 20 & \
      invf inv-b add `cat <_P1` -d "Support" -u hour -q 10 -p 35 -t 20 & \
      invf inv-b add `cat <_P2` -d "Cleanup" -u hour -q 5 -p 45 -t 20 & \
      wait & \
      invf inv download `cat <_P3` -f pdf -o ~/invoices/. & \
      invf inv send `cat <_P4` -f odt -e accountant@email.com
$ rm _P1 _P2 _P3 _P4
```

### Factor and bash inspired Rejy features

#### Pipewords

Pipewords is a word with pipe character on the left. It turns a word referencing a single argument function into it's postfix version. This mimics the stack with depth 1.

    inc inc 10      // returns 12
    10 |inc |inc    // returns 12
    
    print parse-title read http://www.cebelca.biz       // prints title of webpage
    http://www.cebelca.biz |read |parse-title |print    // prints title of webpage

    now |year |inc |print#

#### Opwords

If the stack needs depth of two or more, you can use the opwords. Opwords are words referencing an function of arity 2 or more with typical infix operator character on the left, or dot (.) or single character operators.

    10 + 20 - 5 - 10            // returns 15 ; + - are the opwords that reference an ordinary 2 arg. function
    "Woof" +join "Weef!"        // returns "WoofWeef!" 
    
Opwords can take more than two arguments on the right side.
    
    join3 "Woof" " " "Woof" 
    "Woof" +join3 " " "Woof"     // returns "Woof Woof
    
    set person name "Jane"
    person .set name "Jane"        
    
#### Combining pipewords and opwords

    print either greater? length? block 10 [ "greater" ] [ "lesser" ]
    block |length? .greater? 10 .either [ "greater" ] [ "lesser" ] |print
    
Maybe having two notations (polish + this postfix/infix) will prove to be confusing. Maybe some will program mostly in polish
and other mostly in postfix/infix. We will see.

I want to explore this still, because just polish (prefix) notation is very cumbersome in some cases. Especially if you 
code in a way where you don't want store patial results in a local variable w/o technical reason.

Rebol/Rejy in polish notation can also do inline assignment and debugging very nicely. Not sure about in/post-fix yet.

    a: sum sum 10 e: 20 30      // b is 50, a is 60
    
    20 +sum 30 e: +sum 10 a:    // this can't work as a: at the end can't be distinguisher from being part of a new expression
                                // it we wanted inline assignment in in/post fix it s    

    a: 20 +sum 30 +sum 10       // this works but there is no inline assignemnt possible
    
    e: 20 +sum 30               // you would have to do it in two expressions
    a: e +sum 10
    
One benefit of in/post-fix notation is interactive development in REPL, where you start with initial data and then interactively add transformations to it. Similar to bash. Pipewords are like bash pipes, opwrds are more like bash command xargs.

    > file
    %order.csv
    
    > file |load
    person,item,qty
    Jane Doe,Firestarter,1
    Joe Doe,Wood,33
    Joe Doe,Meat,2
    
    > file |load .parse-csv { ignore header separator "," to blocks }
    { { "Jane Doe" "Firestarter" 33 } ... }
    
    > file |load .parse-csv { ignore header separator "," to blocks } .map 'x [ first x ]
    { "Jane Doe" "Joe Doe" "Joe Doe" }
    
    > file |load .parse-csv { ignore header separator "," to blocks } .map 'x [ first x ] |unique
    { "Jane Doe" "Joe Doe" }
    
QUESTION: Looking at this. Should we just join opwords and pipewords so also .map upthere could be |map?
QUESTION: Rejy has no syntactic sugar and this will probably remain. But there would be an option that "| word" would be turned to "|word" which would make it nicer looking. So far it's NO.

### Other ideas

Modules should be separated from types/classes (object tags in our case). 
    
### LISPs

First ... IANAL (I am not a lisper)

**PROS:** lisp is really solid base homoiconic language, great well tested runtimes, macro system, tons of research and advances, hail lisp!

**CON:** code gives little visual hint about what is going on (for example asignment). It's too "flat". Full homoiconicity doesn't exclude that (look at rebol). Clojure is better in this regard.

**CON** - macro system: you have to think in two separate realities, compile-time and run-time. The forced duality isn't helpful to programmer focusing just on "business logic" IMHO. It sounds like a premature optimisation would.

**CON** - macro system: using macros for making small disperse changes to language might be counterproductive. Changes
to language evaluation only make sense in big important enough cases where learning new customized dialect brings more benefits than just using normal functions.

**PRO** - CLOS, common lisp object system is great

### LISPs inspiration to Rejy

REBOL, which is the core of Rejy - is basically LISP without parenthesis and compile time macros, so that is that. 

Rebol traded parenthesis for fixed arity functions - that's why it doesn't need them and needs fixed arity. Rebol has no compile
step so no compile time macros. You can do every operation Lisp macros do at the runtime though ... it's not always the same 
bacause of performance.

#### CLOS - generic methods

Rejy objects are meant to be data only. And we have generic methods as CLOS, Dylan, Factor. Methods for one "class" could be defined in different modules (vocabolaries). 

Instead of Classes Rejy would use prototypic object system (probably delegated). Complex values would called Tags (instead of classes in CLOS) on which generic methods would dispath on. 
Tags will have it's own dialect that could define validation rules for object slots and default values.

    // STILL FORMING THE IDEAS
    
    // make-tag object and tuple are functions, <tag> is tag-word ... <tag>: is tag-set-word
    
    // define a tag and (optional) validation rules - could the rules also remove the need for constructor?
    // If the rules aren't met produce an error with validation error description as a block (computer readable)
    <person>: make-tag { name: required and string age: optional 0 and integer birth: optional none and iso-date }
    
    // create two objects with tag person
    jim: object <person> { name: "Jim" age: 40 }
    jane: object <person> { name: "Jane" }
    
    // create object w/o Tag
    car: tuple { brand: "Audi" owner: jane }
    // set the tag after object is already created. 
    set-tag car <car>: make-tag { brand: required }
    
    car/owner/name |print
    
    car/owner: jim
    
    introduce: method { <person> obj } { print join "My name is " obj/name } 
    introduce: method { <car> obj } { print join "Vroom Vroom by " obj/brand } 
    
    > { jim jane } |do-each 'x { introduce x }
    My name is Jim
    My nam is Jane
    
    > introduce car
    Vroom Vroom by Audi

Idea is that Tags can also have transformers. These are functions that automatically transform object from one tag to another. This also helps with object versioning / upgrade.

    register-transformer <person> 'to <employee> { name: :full-name birth: code (calc-year age ... )
    // still WIP .. does it even make sense to have them? Are they registered in some register and 
    // automatically used or localy bound to words. Should these be just normal functions ... maybe only make 
    // obj-transformation dialect that helps, convert, combine values from one object to another.
    // THIS DOESN'T MAKE SENSE ... THE FUNCTION DOESN'T KNOW TO WHAT WORD IT WILL BE SET AND WORD DOESN'T
    // HAVE A TYPE / TAG .. the value itself has it.
    
    // this would be just normal generic method
    to-employee: multi { <person> person } { transform { ..... } }
    a-employee: to-employee a-person

### Limits so far

In version pre 0.5 objects can have just one Tag and generic methods dispatch just on first argument.

Rejy has prototypical object model. Objects can have prototype objects. It would be nice if tags could be composed but so far tags don't combine automaticall when prototype is set. The child object has to manually have it's tag set.


### REBOL

If you switch {} with [] Rejy is REBOL. The core is the same. There are additions and changes on top of it. Like infix (opwords) or change in object system (generic words), but Rejy is first and foremost a REBOL's child if not even clone.

One thing about REBOL still remains open for me, and that is scoping. REBOL has a unique scoping mechanism called 'definitional scoping'. There was one document written about it by a REBOL guru Ladislav Mercir. I have to go through it again to really figure out if this is something I want or need to get all the REBOL dynamicism. Maybe it's required for dialects to really work - or thing like user level implementation of CLOSURES inside rebol (implemented by the same Ladislav)

BINDOLOGY:

https://en.wikibooks.org/wiki/Rebol_Programming/Advanced/Bindology

A blogpost about it:

http://blog.revolucent.net/2009/07/deep-rebol-bindology.html

Ladislav has written many more documents exploring advanced REBOL sides and many hard to believe possible functions for rebol - like the clojure one, but they seem lost :( . I see internet never forgets only works for awkward photos.
