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

**PROS:** it works! it's infinitely versatile and extensible! great "repl" (shell), limitles runtime (whole computer / OS)

    cat invs201810.txt | grep -P $"\tpe\t|\tp\t" | cut -f1 | xargs -i ./downloadone invoice-sent {} pdf
    
    cat dls201810.txt | grep -P $"\tpe\t|\te\t" | head -n100  | awk '{xx= "./send-email invoice-sent " $1 " " $3; system(xx)}' > log.txt

**CONS:** it's incoherent, it works on raw text (no structure)

**CONS:** for deeper stack problems you are using named pipes which are somewhat cumbersome

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

Single depth stack problems are solved via **pipewords**. Pipewords are normal functions taking 1 argument that get "|" on left side. Any normal function get's used as pipeword by adding the pipe character in front

    inc inc 10      // returns 12
    10 |inc |inc    // returns 12
    
    print parse-title read http://www.cebelca.biz       // prints title of webpage
    http://www.cebelca.biz |read |parse-title |print    // prints title of webpage

    now |year |inc |print#

#### Opwords

If the pipe or stack needs depth of two, cou can combine in the opwords. Opwords are normal functions with arity 2 or more
that you add a typical infix operator in front (+-\*) and ~. Or single character operators.

    10 + 20 - 5 - 10            // returns 15
    "Woof" +join "Weef!"        // returns "WoofWeef!"
    
Opwords can take more than two arguments

    "Woof" join3 " " "Woof"     // returns "Woof Woof"
    
#### Combininf Pipewords, Opwords, Setwords and more

    block |length? ~greater? 10 ~either [ greater ] [ lesser" ] |print
    // equals 
    print either greater? length? block 10 [ "greater" ] [ "lesser" ]
    
    

### LISPs - homoiconic list based languages

First ... IANAL (I am not a lisper)

**PROS:** lisp is really solid base homoiconic language, great well tested runtimes, macro system, tons of research and advances, hail lisp!

**CON:** code gives little visual hint about what is going on (for example asignment). It's too "flat". Full homoiconicity doesn't exclude that (look at rebol). Clojure is better in this regard.

**CON** - macro system: you have to think in two separate realities, compile-time and run-time. The forced duality isn't helpful to programmer focusing just on "business logic" IMHO. It sounds like a premature optimisation would.

**CON** - macro system: using macros for making small disperse changes to language might be counterproductive. Changes
to language evaluation only make sense in big important enough cases where learning new customized dialect brings more benefits than just using normal functions.

**PRO** - CLOS, common lisp object system is great

