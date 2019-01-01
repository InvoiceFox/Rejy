# Looking at factor, unix shell, lisps

> These are great languages ten levels above my toy interpreter, and people working on them were ten levels above me (some 
> self taught developer). I can still have opinions and write them down, for my sake at least, but have the first scentence 
> in mind when you are reading this.

### Factor - a stack based language

I used to program in factor for about a year. The stack approach certanly has it's magic. Most of it can't be seen from a cursory look. A Factor in particular, was adding very advanced and cool solutions to the mix. 

**PRO:** these languages reject **variable names**. And many times variables are just unnecesary noise that breaks the flow on logic. This is some factor code:

    "Hello, world" print                                            ; prints "Hello, world"
    
    { 4 8 15 } [ 2 * ] map .                                        ; -- puts { 8 16 30 } on stack
    
    "A man, a plan, a canal: Panama." [ Letter? ] filter >lower     ; -- puts amanaplanacanalpanama on stack
    
    "http://factorcode.org" http-get nip string>xml                 ; loads page, turns it into XML nodes
    "a" deep-tags-named                                             ; finds all a tags
    [ "href" attr ] map                                             ; maps list of tags to list of href attrs
    [ print ] each                                                  ; prints each of links

**CONS:** the language introduces stack shuffling words, which is even bigger line noise.

**PRO:** because of this, many advanced solutions vere invented that make the code nicer flowing (see factorcode.org)

**CONS:** with all the advanced solution, language has become to wide

**PRO:** the postfix sytax is naturaly oriented as the operations flows in a program

From the document: http://factorcode.org/littledan/dls.pdf

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
    
    

### LISPS - homoiconic list based languages

IANAL (I am not a lisper)

PROS: lisp is really solid homoiconic (code is data and data is code) language, great well tested runtimes, macro system

CONS: code doesn't give enough visual hints about what is going on (for example asignment). It's too flat. Full homoiconicity doesn't exclude that (look at rebol)

CONS: macro system: you have to think in two separate realities, compile-time and runtime. This duality doesn't help clear thinking about "business logic" IMHO. It sounds like a premature optimisation. 

CONS: macro system: macros for doing small disperse changes to language incurs more cost (of surprise) than it's worth. Changes
to language evaluation only makes sense in big important enough cases where learning new specialised dialect makes sense over just using language as it is.

