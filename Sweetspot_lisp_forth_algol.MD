# lisp, forth and algol

    This could be horribly pretentious text. I am aware of that, you reader - please be aware of that too.
    I am nobody, but I can still type, and you can ignore ...

# Line noise

Let's say each of these languages removes one kind of line noise and introduces another.

### FORTH / Factor - Concatenative languages

* **PRO:** these languages reject **variable names**. And many times variables are just unnecesary noise that breaks the flow on logic.

    "Hello, world" print                                            ; prints "Hello, world"
    
    { 4 8 15 } [ 2 * ] map .                                        ; -- puts { 8 16 30 } on stack
    
    "A man, a plan, a canal: Panama." [ Letter? ] filter >lower     ; -- puts amanaplanacanalpanama on stack
    
    "http://factorcode.org" http-get nip string>xml                 ; loads page, turns it into XML nodes
    "a" deep-tags-named                                             ; finds all a tags
    [ "href" attr ] map                                             ; maps list of tags to list of href attributes
    [ print ] each                                                  ; prints each of links

* **CONS:** the language introduces stack shuffling words, which is even bigger line noise.
* **PRO:** because of this, many advanced solutions vere invented that make the code nicer flowing (see factorcode.org)
* **PRO:** the postfix sytax is naturaly oriented as the operations flows in a program

    10 a b c d          ; -- factor code
    
    d(c(b(a(10))))      ; -- python code
    
    (d (c (b (a 10))))  ; -- lisp code
    
    d c b a 10          ; -- rebol code
    
    d c b a 10          ; -- Rejy code
    10 |a |b |c |d      ; -- Rejy code also using pipewords

* **CONS:** factor also introduces local variables, but they get visually

Rejy learns from all this and offers few features.

Great document to read about factor: http://factorcode.org/littledan/dls.pdf


### LISPS - homoiconic list based languages



    
